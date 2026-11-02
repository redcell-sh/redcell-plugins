---
name: linux-privesc
description: Linux local privilege-escalation playbook for authorized hosts — post-shell enumeration checklist, credential hunting, sudo/SUID/capabilities/cron abuse, arbitrary-file-write escalation primitives, and last-resort kernel exploits. Use after gaining any Linux foothold when you need root.
metadata:
  short-description: Linux foothold → root playbook
  category: exploitation
---

# Linux Privilege Escalation

> Authorized engagements only (see `authorized-use`).

## 1. Post-shell enumeration (run the whole checklist)
- `id`, `sudo -l`, `uname -a`, OS release, PATH, and environment.
- Cron and timers: `/etc/crontab`, per-user crons, systemd timers — anything executed by root that
  you can influence.
- SUID/SGID binaries (`find / -perm -4000 -type f 2>/dev/null`) — cross-check each against
  GTFOBins-style abuse patterns.
- File capabilities (`getcap -r / 2>/dev/null`), writable scripts/configs invoked by root,
  world-writable directories early in root's PATH.
- Local-only services (`ss -tlnp`) and their default credentials.
- Run an enumerator (linpeas/lse) for breadth, but manually verify every hit before acting.

## 2. Credential hunting (the most common real chain)
- Application configs: database connection strings, `.env` files, deployment scripts, backups,
  service configs under `/var` and `/opt`.
- Histories: current user's, other users', and service accounts' shell histories.
- SSH keys: current user, other users, root; writable `authorized_keys`; keys with passphrases worth
  cracking.
- Process arguments (`ps auxww`), environment variables, cloud instance metadata, tokens in files.
- Captures and memory: PCAPs left on the box; Java application **heapdumps** — credentials frequently
  sit in plaintext in memory dumps.

## 3. Escalation primitives
- **sudo misconfigurations** (NOPASSWD entries, command wildcards, allowed binaries): abuse per the
  GTFOBins pattern for that binary — pager escapes (`journalctl`-style), interpreter shells
  (`dotnet`, `awk`, scripting runtimes), and world-writable scripts run via sudo are the classics;
  check the sudo version for known bypasses.
- **Cron/task abuse**: writable scripts or directories executed by root; PATH manipulation against
  scripts that call bare command names.
- **Configuration-management abuse**: automation that runs as root — Ansible playbooks/tasks and
  equivalents — is a code-execution primitive if you can write anywhere in its config tree; audit
  the full task/template list for injection points.
- **Log-handling races**: log rotation/movement over paths you can influence can be raced into
  following a symlink; match the exact tool version to its known race before attempting.
- **Arbitrary file write** (as any user, into a privileged path) → write where root will execute:
  cron scripts, systemd units, `authorized_keys`, or overwrite a binary a root service runs.
- **Service-specific exploits**: confirm the exact version first, then the matching local
  privilege-escalation bug.
- **Kernel exploits last**: check `uname -a` against known escalation bugs, but treat them as a
  final option on production — they are loud and can crash hosts.

## 4. Reuse and pivot
- Every credential found gets tried everywhere: as another local user, over SSH, against local
  databases, and on internal web apps. Real chains go user → user → root far more often than a single
  magic bug.

## 5. Verify and document
- Prove root (`id`), capture the exact chain, and clean up: remove dropped files, revert modified
  configs, delete any added cron entries or keys.

---
*Methods distilled clean-room from public community lab writeups (0xdf hacks stuff, IppSec, and
similar public walkthrough material). No writeup text, flags, or target-specific answers reproduced.*
