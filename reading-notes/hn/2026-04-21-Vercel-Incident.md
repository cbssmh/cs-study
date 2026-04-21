# Vercel Incident (April 2026)

## Summary

A multi-step security incident where small, unrelated weaknesses combined into a major breach:

```
Roblox cheat download
→ Lumma infostealer infection
→ Context.ai account compromise
→ OAuth access to Vercel employee Google Workspace
→ Vercel environment variables exposed (plaintext)
→ Data exfiltration + ransom attempt
```

---

## Core Concept

Supply chain attack as a compositional failure.

Each step alone was low impact, but the chain created a system-level failure.

---

## Key Failures

### 1. OAuth over-permission

* Third-party AI tool had broad Google Workspace access
* “Allow All” permission model
* No practical granularity
* Users approve without reviewing scope

---

### 2. Unsafe defaults (Vercel)

* Environment variables stored in plaintext by default
* “Sensitive” flag required manual opt-in
* Most users assumed encryption at rest

→ mismatch between system behavior and user expectation

---

### 3. Human factor

* Untrusted software installed
* Permissions granted without verification
* Security assumptions not validated

---

### 4. Trust chain amplification

* Multiple SaaS tools connected via OAuth
* AI tools require broad access by design
* One compromised node propagated across systems

---

## Key Insights

### Convenience vs security

Convenience dominates decision-making until failure occurs.

Secure alternatives exist but require more setup and maintenance.

---

### Secure-by-default

Users do not rely on documentation.

Default behavior defines real security posture.

---

### System = trust graph

Modern systems are composed of:

* SaaS services
* OAuth integrations
* shared credentials

→ implicit trust accumulates across nodes

---

### AI tools as attack surface

AI tools expand access scope:

* email
* documents
* codebases

→ increase blast radius of compromise

---

## Practical Takeaways

### Secrets management

* Do not rely on platform UI flags
* Use dedicated secret systems:

  * AWS Secrets Manager
  * HashiCorp Vault
  * SOPS + encryption

---

### OAuth hygiene

* Minimize permissions
* Audit authorized apps regularly
* Remove unused integrations

---

### System design

* Assume third-party compromise
* Reduce implicit trust boundaries
* Isolate critical components where possible

---

### Personal rule

Spend more time reviewing permissions than granting them.

---

## One-line takeaway

Convenience increases attack surface.

---

## Notes

* Failure was not due to a single exploit

* Caused by interaction between:

  * human behavior
  * default configuration
  * system connectivity

* Representative example of modern SaaS risk

---

## Tags

#security #system-design #oauth #supply-chain #case-study
