# 🔗 Complete Investigation Chain

The entire investigation can be summarized as:

                 Pantalones Leak Site
                         │
                         ▼
                AetherFlow Archive
                         │
                         ▼
                   Hidden .exfil.sh
                         │
                         ▼
              Attacker Panel + API
                         │
                         ▼
                  API Enumeration
                         │
                         ▼
                Internal Messages
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
       OPSEC mistakes         Credential sharing
                                     │
                                     ▼
                              Base64 encoded
                                     │
                                     ▼
                               Decode password
                                     │
                                     ▼
                              Admin Panel Login
                                     │
                                     ▼
                                   FLAG

                                   
# 🧠 Key Findings
## Finding 1 — Sensitive information exposed in a script

The attackers published an operational script containing information about
their infrastructure.

Security impact:

An attacker or researcher could potentially use this information to locate
and interact with the management API.

## Finding 2 — Authentication secrets were exposed

The leaked script contained an authentication mechanism for the API.

Security impact:

Exposure of authentication material could allow unauthorized access to
attacker infrastructure.

## Finding 3 — Credentials were shared through chat

The attackers shared a password through their internal messaging system.

Security impact:

Anyone gaining access to the conversation could potentially obtain the
credential.


## Finding 4 — Base64 was incorrectly treated as protection

The password was Base64 encoded rather than securely protected.

Security impact:

Base64 provides no meaningful confidentiality.

## Finding 5 — Administrative access was protected by weak credential practices

The discovered credential could be used to authenticate to the CTF's
administrative interface.

Security impact:

Credential reuse and poor secret management can turn a single information
leak into administrative compromise.

# 🛡️ SOC Analyst Perspective

- The biggest lesson I took from this CTF was the importance of evidence correlation.

- Initially, the individual pieces of information didn't appear to be related

- But when the evidence was correlated, it formed a complete attack chain.

- This is similar to real SOC investigations.

- A single alert may not tell the entire story.

An analyst may need to correlate:

- File activity
- Authentication events
- Network connections
- Process execution
- Credentials
- Emails
- Logs
- Threat intelligence
- User activity


# 🔍 What I Would Look For During a Real Incident

This CTF changed the questions I would ask when investigating a
compromised environment.

Instead of only asking:

"What exploit was used?"

I would also ask:

"What did the attacker leave behind?"

For example:

- **Files**
  - `*.log`
  - `*.bak`
  - `*.old`
  - `.*`
  - `.env`
  - Configuration files
  - Scripts
  - Temporary files

- **Credentials & Secrets**
  - Passwords
  - API keys
  - Tokens
  - SSH keys
  - Cloud credentials
  - Database credentials

- **Infrastructure**
  - IP addresses
  - Domains
  - API endpoints
  - Command-and-control infrastructure
  - Management panels

- **Communications**
  - Internal chat
  - Emails
  - Ticket systems
  - Notes
  - Comments

These artifacts can provide valuable context during an investigation.

# 📚 Lessons Learned
1. Hidden files are not secure files

A dotfile can still be discovered and analyzed.

2. Base64 is not encryption

Sensitive information should never rely on Base64 for protection.

3. Never share passwords through chat

Use a secure secrets-management solution instead.

4. Don't hard-code secrets into scripts

API keys, tokens, passwords, and authentication headers should be managed
through appropriate secret-management mechanisms.

5. Credential reuse increases blast radius

A compromised password becomes much more dangerous when reused across
multiple systems.

6. OPSEC failures can expose sophisticated attackers

Even technically capable attackers can create serious security weaknesses
through simple operational mistakes.

7. Correlation matters

The flag wasn't obvious from the beginning.

It required connecting multiple pieces of evidence.

# 🛠️ Tools Used
- CyberChef
- Web browser
- Source-code inspection
- HTTP/API analysis
- Manual investigation
- Base64 decoding


# 🏆 Challenge Status
Pantalones CTF Status: COMPLETED ✅
Flag: 
```
flare{pantal0n3s_g0t_pantsed_2026}
```
