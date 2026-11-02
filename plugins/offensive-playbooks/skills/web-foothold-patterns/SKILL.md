---
name: web-foothold-patterns
description: Pattern library for turning web application access into a shell or foothold in authorized engagements — surface expansion (subdomains/vhosts/hidden panels), content discovery and source disclosure, upload-to-RCE abuse, LFI/file-read to credential discovery, deserialization and template injection, image-tool exploitation, and SSH key recovery. Use whenever a target's web surface is the likely entry point or an HTTP service needs deeper attack.
metadata:
  short-description: Web entry → shell pattern library
  category: web
---

# Web Foothold Patterns

> Authorized engagements only (see `authorized-use`). These are the recurring ways community
> writeups actually convert a web surface into a shell — generalized patterns, not target-specific
> recipes. The class taxonomy (SQLi, XSS, SSRF, IDOR) lives in `web-app-testing`; this skill is
> about the *foothold*.

## 1. Expand the surface before attacking the page
- Subdomain fuzz (`ffuf`/`gobuster` on DNS, watch for wildcard responses) — hidden **dev/staging
  panels** are frequently the real entry point, not the main site.
- Enumerate vhosts, ports, and API surfaces; revisit each after learning the company/project names
  found in page content and JS.
- Crawl authenticated and unauthenticated; extract endpoints, parameters, and JS bundles before
  choosing an attack.

## 2. Source disclosure and file-read primitives
- A file read / LFI / path traversal is worth more than a flag: use it to pull **application source,
  configs, and service files** — SSH keys, database credentials, and internal connection strings
  usually follow.
- Check for exposed `.git` directories, backups, `debug` endpoints, and swagger/openapi docs.
- With source in hand, read the logic: how the app authenticates, connects to backends, and handles
  uploads — the intended bug is usually visible in the code.

## 3. Upload → code execution
- Known-CMS/plugin file-upload bugs first (image galleries, blog plugins): an authenticated upload
  that reaches a web-accessful directory often means direct RCE.
- Filter bypass patterns: extension allowlist confusion (`php5`, double extensions), content-type
  and magic-byte checks, null-byte truncation, archive nesting (zip-of-zip extraction quirks).
- On IIS/ASP.NET stacks, remember config files (`web.config`) can themselves become execution
  vehicles; on LAMP stacks, `.htaccess` plays the same role.

## 4. Credentials from reads, reads from credentials
- SSH private keys recovered via file read → offline passphrase cracking (`ssh2john` + hashcat) →
  shell as the key's owner. This chain appears constantly in real chains.
- Reuse every recovered credential: web logins, SSH, database connections, and internal panels.

## 5. Deserialization and template injection
- Identify the stack, then aim the matching primitive: .NET (JSON/binary gadget chains, often
  found by reversing a web-reachable DLL), Java, PHP, Python (pickle), Ruby.
- Expression injection: SpEL (Spring), SSTI (Jinja/Twig/Freemarker), EL — confirm the engine, then
  the documented sandbox-escape pattern for that exact version.
- Enumerate non-HTTP channels too: websockets and localhost-bound services often expose the same
  app logic without the UI's protections — port-forward when reachable.

## 6. Image and document processing abuse
- File-handling binaries have version-specific RCEs (exiftool-style parser bugs): fingerprint the
  exact version before attempting one.
- Command-injection in converters/thumbnailers (mogrify-style tooling) via crafted filenames,
  profiles, or config directives.

## 7. Public exploit adaptation
- Version match → public PoC. Always read the PoC first, adjust paths/targets/payload format, and
  prefer the scriptable form over a framework module when you need control.
- Classic network-service CVEs (old SMB/FTP/CGI daemons) still open modern engagements — never skip
  the version lookup just because the service is old.

## 8. After the shell
- Confirm the app context (`www-data`/service account), then route to the OS escalation playbook
  (`linux-privesc` / `windows-privesc`).
- Keep the exact request(s) that produced the shell — they are the core of the report
  reproduction.

---
*Patterns distilled clean-room from public community lab writeups (0xdf hacks stuff, IppSec, and
similar public walkthrough material). No writeup text, flags, or target-specific answers reproduced.*
