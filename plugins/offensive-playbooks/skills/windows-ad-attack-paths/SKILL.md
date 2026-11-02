---
name: windows-ad-attack-paths
description: Playbook for Windows and Active Directory attack paths in authorized environments — null-session and LDAP recon, policy-aware credential spraying and validation, BloodHound path analysis, Kerberos attacks, ADCS template abuse, and privilege abuse to domain dominance. Use whenever a target presents Windows/AD services (SMB, LDAP, Kerberos, WinRM) or when domain credentials are found.
metadata:
  short-description: Windows/AD attack-path chaining playbook
  category: active-directory
---

# Windows / AD Attack Paths

> Authorized engagements only (see `authorized-use`). Ordered the way community writeups actually
> chain these attacks: recon → credentials → validate → map → abuse → dominance.

## 1. Domain recon
- Ports 53/88/135/389/445/464/636/3268/9389 plus high RPC ports → a Domain Controller.
- Resolve names first: `netexec smb <ip> --generate-hosts-file` → merge into `/etc/hosts`. Every later
  step (Kerberos, LDAP, WinRM, ADCS) depends on correct hostnames and the domain name.
- Try null and guest sessions (`netexec smb -u '' -p ''`); RID-brute for usernames; anonymous LDAP
  queries (`(objectClass=*)`, `(sAMAccountName=*)`) often expose users, groups, OUs, and
  password-bearing description fields.
- Kerberos is time-sensitive: if nmap reports clock skew, sync time (`ntpdate`) **before** any
  Kerberos authentication, or tickets will fail for reasons that look like wrong credentials.

## 2. Credential attacks (policy-aware)
- Read the **password policy before spraying** (`--pass-pol`): lockout threshold, complexity, history.
  Spray within the policy — or not at all. Account lockouts damage the engagement.
- Take every found credential and **validate it against every protocol**: SMB, WinRM, LDAP, SSH, web
  logins. In netexec-style tooling, `Pwn3d!` marks a credential that yields a shell.
- Reuse tests: same password across users; same user across protocols and hosts; service accounts on
  multiple machines.
- Recover more credentials: crack captured hashes offline (`hashcat`); read PCAPs, configs, memory
  dumps, and directory descriptions; file-share icon poisoning (crafted `.scf`/`.url` files to
  capture NTLM handshakes) only when clearly in scope.
- Registry-stored secrets are classic hops: autologon credentials, VNC/remote-tool passwords, and
  sync-service secrets (Azure AD Connect-style) once RPC/WinRM access exists.
- Crack password safes and encrypted app configs — the key frequently sits in a nearby file or
  field of the same configuration.
- Password-change chains: `ForceChangePassword`-class rights let you reset another account's
  password and keep the original hash — follow BloodHound edges hop by hop instead of hunting for
  one big bug.

## 3. Map the attack paths
- Collect with **BloodHound** (or netexec's collection) at every new credential tier, and query
  *shortest paths* from the current user to Domain Admins and other high-value targets.
- Classic edges to look for: group memberships, `GenericAll`/`WriteDacl`/`WriteOwner`/`GenericWrite`
  on users/groups, constrained/resource-based delegation, `AddMember`/`AddSelf`, machine-account
  quota abuse.
- **Targeted Kerberoasting** of SPN-bearing accounts (crackable service passwords); AS-REP roastable
  accounts (no preauth) when no credentials exist yet.
- Machine accounts (`$` accounts) are credentials too: shadow credentials, quotas, and
  resource-based constrained delegation all pivot through them.
- **gMSA passwords** are readable by allowed principals (or via a sync service's credential store) —
  a quiet, domain-wide credential pivot that skips cracking entirely.
- **DnsAdmins** membership → load an attacker-controlled DLL into the DNS service (SYSTEM).
- Legacy **Exchange** groups and their inherited ACLs still create powerful write-DACL chains; let
  BloodHound find the shortest route rather than assuming.

## 4. ADCS / certificate abuse
- Whenever a CA is advertised, enumerate certificate templates via LDAP/`certipy find`.
- Match findings to the standard misconfiguration classes (ESC1–ESC16): SAN-enabled enrollment by
  low-privilege users, enrollment-agent chains, vulnerable template ACLs and schema issues. Identify
  the class first, then build the matching attack.
- A forged client-auth certificate is effectively a credential for the named user — validate it,
  then use it like one.

## 5. Privilege abuse at each tier
Run `whoami /priv` after every shell; each enabled privilege maps to a named technique:
- **SeImpersonate** → potato-family abuse from service-account shells.
- **SeBackup/SeRestore** (Backup Operators) → copy locked hives: `reg.py`-style saves for SAM/SYSTEM,
  then a `diskshadow` shadow copy plus `robocopy /b` for `NTDS.dit`; crack offline with
  `secretsdump.py` (local SAM first, then the domain database).
- **SeTakeOwnership/SeRestore** → take ownership of a binary that a SYSTEM service executes.
- **SeDebug** → inspect/impersonate privileged processes.
- **AD Recycle Bin** membership → read deleted objects (old credentials, service accounts).
- Driver-based token theft is high-impact and high-risk: only with explicit authorization.

## 6. Domain dominance and proof
- With DA-equivalent rights: full domain hash extraction (`secretsdump.py`), golden ticket or golden
  certificate, or DCSync. Validate each hop (`netexec`, WinRM, `evil-winrm`) before claiming it.
- Never stop at "it should work" — reproduce and capture it.
- Document the complete chain hop by hop; writeups and engagement reports alike are judged on the
  full path, not the endpoint.

---
*Methods distilled clean-room from public community lab writeups (0xdf hacks stuff and similar
public AD walkthrough material). No writeup text, flags, or target-specific answers reproduced.*
