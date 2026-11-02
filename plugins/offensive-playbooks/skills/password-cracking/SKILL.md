---
name: password-cracking
description: Offline password and hash cracking methodology for authorized engagements — hash format identification, wordlist and rule selection, mask and mutation attacks, cracking Kerberos tickets, SSH keys, password safes, VNC passwords, and encrypted configs, and knowing when to stop. Use whenever recovered hashes, tickets, keys, or encrypted credentials need to be cracked offline.
metadata:
  short-description: Offline credential cracking playbook
  category: password-cracking
---

# Password Cracking

> Authorized engagements only (see `authorized-use`): crack only material recovered from in-scope
> targets, offline, on your own hardware. Cracking is usually the cheapest path to the next hop —
> budget real time for it.

## 1. Identify the exact format before anything else
- Use format detectors (hashid, `hashcat --show`-style tooling, netexec's module output) so you
  don't burn GPU hours on the wrong mode. NTLM vs NetNTLMv2 vs Kerberos tickets vs bcrypt have
  wildly different costs.
- Common lab/engagement formats: NTLM (1000), NetNTLMv2 (5600), Kerberoast (13100), AS-REP
  (18200), bcrypt (3200), SHA512crypt (1800).

## 2. Wordlist and rule strategy
- Baseline: rockyou with `best64`/`dive`-class rules; then organization-specific mutations —
  company name, service names, years, seasons, sports — combined into a small generated list.
- **Username-derived mutations** (user + digits/symbols) crack service accounts constantly.
- Mask attacks (prefix/suffix digits/punctuation) after the dictionary passes; keep masks informed
  by any password policy observed in the target (`--pass-pol`).

## 3. Special formats worth knowing cold
- **Kerberos tickets** (Kerberoast/AS-REP): service accounts carry human-style passwords; a single
  cracked ticket is often a domain-wide foothold.
- **SSH private keys**: extract to crackable format (`ssh2john`), then passphrase-crack; the key is
  frequently reused across users and hosts.
- **Password safes** (KeePass, Password Safe): the vault password is usually reused from elsewhere,
  or crackable from the policy you already know; vault contents are engagement gold — dump them
  minimally and document.
- **VNC/remote-tool passwords**: registry-stored values use fixed keys — identify the tool and use
  its documented decoder.
- **Encrypted app configs**: the encryption key frequently sits in the same config or an adjacent
  file; recover the key first, not the ciphertext brute-force.

## 4. Feed results back into everything
- Every cracked credential gets validated across all protocols and reused as a spray candidate
  (see `windows-ad-attack-paths`); password patterns found once (e.g., `Season+Year!`) become
  mutation rules for the *other* hashes in the pile.

## 5. Know when to stop
- Time-box pure brute force; if a hash resists targeted dictionaries, move to other paths
  (credential hunting, misconfigurations) and revisit with better context.
- Record what was attempted (wordlists, rules, runtime) — negative results still inform the report.

---
*Patterns distilled clean-room from public community lab writeups, where cracked tickets, SSH keys,
safes, and reused credentials are the most common hop between stages. No writeup text or flags
reproduced.*
