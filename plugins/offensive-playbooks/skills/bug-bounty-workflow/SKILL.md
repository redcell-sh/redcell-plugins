---
name: bug-bounty-workflow
description: End-to-end authorized bug bounty workflow — program scope discipline, asset and subdomain recon, automated lead generation, manual validation with an oracle, deduplication, and report writing. Use when hunting on a bug bounty platform, VDP, or any authorized public-facing program.
metadata:
  short-description: Bug bounty recon → validate → report workflow
  category: web
---

# Bug Bounty Workflow

> Authorized targets only: the program policy is the scope document (see `authorized-use`). The
> vulnerability-class taxonomy lives in the `web-app-testing` skill; this is the hunting workflow
> around it.

## 1. Scope discipline
- Read the full policy before touching anything: in-scope assets and wildcards, explicit exclusions,
  prohibited actions (typically: no DoS, no destructive testing, no social engineering, no mass
  automated scanning beyond granted tiers, no real-user data exfiltration).
- Anything not listed in scope is out of scope — including interesting neighbors you discover while
  testing.
- Note the reward/impact table: it tells you which findings actually matter to the program.

## 2. Asset recon (passive first)
- Expand the surface from seed domains: certificate transparency (crt.sh), DNS records, subdomain
  enumeration (subfinder/amass), wayback machine, JS analysis, and code-search dorks for leaked
  endpoints, keys, and internal hosts.
- Probe what is alive (httpx), screenshot and categorize: login pages, admin panels, uploads, APIs,
  old frameworks, staging leftovers.
- Port-scan only within policy limits; use ASN/IP-range lookups when infrastructure is in scope.
- Build a ranked target inventory; prefer likely-neglected surfaces (old subdomains, staging,
  forgotten panels) — they carry the best finding-per-effort ratio.

## 3. Automated triage — leads, not findings
- `nuclei` for known templates/CVEs/misconfigurations; `ffuf` for content discovery; JS endpoint
  extraction for hidden APIs.
- Treat every automated hit as a **lead requiring manual confirmation**. Scanner output is not a
  reportable finding and never a report by itself.

## 4. Manual validation (oracle discipline)
- Reproduce with a minimal, deterministic request; distinguish real effects from cache artifacts,
  out-of-scope hosts, and scanner false positives.
- Demonstrate **concrete impact** — what an attacker gains — not a reflected string or a version
  number.
- Stay minimal: use the least intrusive payload that proves the issue; use a program-owned test
  account; stop at the smallest data sample that demonstrates it.
- Respect WAFs and rate limits; throttle tools; never take a service down or degrade it for users.

## 5. Deduplicate and research
- Search the platform for existing reports on the same asset/issue class before writing yours.
- Check the target's stack against public writeups and CVEs — a known vulnerability may already be
  reported; your added value is the confirmed in-scope reproduction and impact assessment.

## 6. Report
- Follow the `report-writing` skill structure: neutral title, severity, affected asset, minimal
  reproduction steps, impact, remediation with CWE/OWASP references.
- Include exactly what a triager needs to reproduce — no unreduced tool dumps.
- Follow the program's disclosure rules, respond to triage follow-ups promptly, and keep the report
  updated as the issue gets fixed.

## 7. Operate like a professional
- Keep a per-program hunting log (targets, tests, findings, dead ends). Patterns recur across
  programs: forgotten subdomains, outdated libraries, IDOR on secondary apps, business-logic race
  conditions.
- Study public bug-bounty writeups the way lab writeups are studied: what was tested, in what order,
  and what made the finding reportable.

---
*Verification discipline and workflow shape distilled clean-room from public community lab and
bug-bounty writeup material. No writeup text reproduced.*
