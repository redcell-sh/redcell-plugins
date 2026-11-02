---
name: windows-privesc
description: Windows local privilege-escalation playbook for authorized hosts — post-shell enumeration, service abuse, unquoted paths, registry secrets, and privilege-token abuse (impersonate/backup/restore/ownership). Use after gaining any Windows foothold when you need SYSTEM on a single host.
metadata:
  short-description: Windows foothold → SYSTEM playbook
  category: exploitation
---

# Windows Privilege Escalation

> Authorized engagements only (see `authorized-use`). Domain-attack chaining lives in
> `windows-ad-attack-paths`; this playbook covers single-host elevation to SYSTEM.

## 1. Post-shell enumeration (run all of it)
- `whoami /all` — user, groups, and privileges. `whoami /priv` is the single highest-signal command on
  Windows: every enabled privilege maps to a named technique.
- `systeminfo`, installed patches, and exact versions of installed software (uninstall registry keys,
  Program Files) → known local privilege-escalation exploits.
- Services: non-default services, **unquoted binary paths containing spaces**, weak service ACLs
  (writable service directories/binaries/configs), services running as SYSTEM that execute anything
  you can influence.
- Scheduled tasks, startup folders, `AlwaysInstallElevated` registry keys, and autologon credentials
  in the registry. Enumeration tools (winpeas and peers) widen coverage — verify every hit manually
  before acting on it.

## 2. Credential hunting before exploits
Real chains escalate with credentials more often than exploits:
- User files and documents, application config files, connection strings, scripts, and backups.
- Saved credentials: Windows credential store (`cmdkey`), DPAPI blobs, RDP connection files, browser
  stores.
- Other users' processes and their command lines; passwords in service configs and scheduled-task
  actions.
- `runas` with any recovered credential — the cheapest user-switch on Windows; test recovered creds
  against every account present on the host before hunting a new bug.
- Crafted documents/shortcuts placed where other users open them can capture authentications (NTLM)
  or execute under their session — only with explicit authorization covering user interaction.
- Memory extraction (LSASS, heapdumps) only with explicit authorization for exactly that.

## 3. Privilege → technique mapping
- **SeImpersonate / SeAssignPrimaryToken** → potato-family abuse from service accounts.
- **SeBackup / SeRestore** → copy locked files (SAM/SYSTEM/NTDS): registry-hive saves, shadow copy plus
  `robocopy /b`, then offline hash extraction (`secretsdump.py`-style tooling).
- **SeTakeOwnership / SeRestore** → take ownership of a binary a SYSTEM service runs, or of a config
  that will be loaded privileged.
- **SeLoadDriver** → driver-loading abuse (authorized engagements only).
- **SeDebug** → inspect and impersonate privileged processes.
- Group memberships matter: Backup Operators, Print Operators, and Server Operators each carry
  documented abuse paths — enumerate `net localgroup` memberships explicitly.

## 4. Exploit only after misconfigurations
- Kernel exploits are the last resort on production hosts, not the first attempt. Prefer
  configuration and credential paths first — they are quieter, more reliable, and better for reports.
- Check `systeminfo` against known local exploits before considering a kernel bug.
- Read any exploit before running it; understand what it changes, and prefer reversible techniques.

## 5. Verify and document
- Prove SYSTEM (`whoami` → `NT AUTHORITY\SYSTEM`) and capture the exact chain.
- Record what each step changed, clean up dropped tools and any modified services/configs, and hand
  the chain to reporting with the evidence attached.

---
*Methods distilled clean-room from public community lab writeups (0xdf hacks stuff, IppSec, and
similar public walkthrough material). No writeup text, flags, or target-specific answers reproduced.*
