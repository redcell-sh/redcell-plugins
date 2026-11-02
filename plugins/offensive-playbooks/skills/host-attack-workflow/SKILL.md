---
name: host-attack-workflow
description: Universal workflow for attacking a single authorized host or lab target — full-surface port scanning, service-by-service enumeration, foothold hunting, post-foothold host enumeration, and escalation. Use when starting work on any standalone target, lab machine, or single-host engagement and you need the scan-to-proof loop.
metadata:
  short-description: Scan → enumerate → foothold → escalate loop
  category: exploitation
---

# Host Attack Workflow

> Authorized engagements only — confirm scope first (see the `authorized-use` skill). This is a
> transferable method loop distilled from how experienced testers work through public lab writeups;
> it applies to any single authorized host or standalone engagement target.

## 1. Map the full surface before anything else
- **All TCP ports first, fast**: `nmap -p- --min-rate 5000 <ip>`. One missed high port wastes hours.
- **Deep-scan found ports only**: `nmap -p <ports> -sCV <ip>` for versions and default scripts.
- Add **top-UDP** only when TCP yields little or the service mix hints at it (SNMP, TFTP, IPsec).
- Record OS hints: TTL (~127 suggests a one-hop Windows host, ~63 a one-hop Linux host), SMB/HTTP
  banners, nmap guesses. Track them; they steer later steps.
- Put any hostname/domain surfaced (certificates, SMB, LDAP, vhosts) into `/etc/hosts` immediately —
  most exploitation tooling depends on names resolving correctly.
- Keep structured per-target notes: ports, versions, credentials, shells, and what failed. Notes are
  part of the deliverable, not an afterthought.

## 2. Enumerate service-by-service, deeply
Work each open service toward exhaustion before moving on. Writeup-grade results come from
enumeration depth, not 0days.
- **HTTP(S)**: visit every port; fuzz subdomains and vhosts (`ffuf`, DNS-wildcard aware) — hidden
  dev/staging panels are frequently the real entry point; check robots, sitemaps, page comments,
  JS bundles, and APIs; run content discovery (`ffuf`/`gobuster` with SecLists); walk login flows and
  file-upload points. Note the exact framework and plugin/theme versions.
- **SMB/RPC**: null and guest sessions, share listing, RID brute; with any creds, dump users, groups,
  and SAM targets.
- **LDAP**: anonymous binds and base queries first (`(objectClass=*)`, `(sAMAccountName=*)`); read
  user/group OUs and descriptions — they often carry credentials.
- **DNS**: zone transfers, subdomain enumeration, host records.
- **FTP / databases / mail / everything else**: anonymous access, default credentials, and
  version-specific known issues.
- For every **exact version string**, check known vulnerabilities/CVEs, default credentials, and
  vendor documentation before inventing your own attack.

## 3. Hunt the foothold
Prefer the least destructive path that works, roughly in this order:
1. **Leaked or default credentials** from banners, LDAP fields, page text, config files, and backups.
2. **Known-vulnerable versions** of exposed applications → public exploit/PoC. Read the exploit code
   before running it; understand what it does and adjust it to the target.
3. **Web logic bugs**: upload-filter bypass (extension, MIME, null-byte, archive nesting), injection,
   SSRF, deserialization, auth bypass.
4. **File-read/LFI primitives**: use them to pull application source, configs, and SSH keys —
   credentials recovered this way often carry the whole chain (see the `web-foothold-patterns`
   skill).
5. **Service misconfigurations**: anonymous access to sensitive shares, permissive services, parser
   quirks.
- If nothing lands, re-enumerate rather than escalate noise: more wordlists, more vhosts, revisit
  earlier services with anything new you learned.

## 4. Post-foothold: stabilize and re-enumerate
- Get a stable interactive shell; record the working directory and privilege level (`id`,
  `whoami /all`).
- Hunt credentials immediately: application configs, shell histories, environment, database dumps,
  memory/heapdumps, packet captures, and other users' files.
- Map what the foothold can reach: network segments, internal apps, databases, domain resources.
- Enumerate services bound to localhost only (internal APIs, admin panels, databases) and
  port-forward to them when they look weaker than the front door.

## 5. Escalate
- Route by OS: the `linux-privesc` playbook, or `windows-privesc` / `windows-ad-attack-paths`.
- The most common real-world chain shape is: foothold user → credential reuse on another
  service/user → privileged group membership → host or domain dominance. Always test credentials
  everywhere before hunting a brand-new bug.
- When multiple escalation paths exist, prefer the intended one (least fragile), but document others
  you found — extra paths are findings too.

## 6. Verify, prove, document
- Prove the access level: `whoami`/`id`, the target's proof object, or a named sensitive asset.
- Re-run the successful chain cleanly once to confirm it reproduces; note any one-shot quirks.
- Write the kill chain as a narrative (recon → foothold → escalation) so the reporting phase (see
  `report-writing`) gets the full path, not just the endpoint.

## Loop discipline
- Stuck? Return to step 2 with everything learned; re-enumerate earlier services with new credentials.
- A "second path" check at the end often reveals both intended and unintended solutions and sharpens
  your methodology for the next target.

---
*Methods distilled clean-room from public community lab writeups (0xdf hacks stuff, IppSec, and
similar public walkthrough material). No writeup text, flags, or target-specific answers reproduced.*
