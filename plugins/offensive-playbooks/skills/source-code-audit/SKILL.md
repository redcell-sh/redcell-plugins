---
name: source-code-audit
description: "Source-code auditing methodology for authorized engagements — reviewing target application source (client-provided repos, recovered source, or file-read access) for security bugs: entry-point mapping, taint tracking to dangerous sinks, authentication/authorization review, secret and dependency scanning, and git history mining, ending in validated findings with file/line references. Use whenever source code for a target is available and needs a security review."
metadata:
  short-description: Target source-code audit playbook
  category: code-security
---

# Source Code Audit

> Authorized engagements only (see `authorized-use`). Available source comes from the client's
> repos, authorized downloads, or file-read primitives during testing — audit what you're given,
  and confirm you may keep copies of what you copy.

## 1. Map the application before hunting bugs
- Identify language, framework, and entry points: routes/controllers, background jobs, message
  handlers, deserializers, and anything parsing untrusted input.
- Trace the deployment shape (config files, environment templates) — misconfigurations are
  findings too.

## 2. Follow untrusted data to dangerous sinks
- Track request parameters, uploads, headers, and IPC input from source to sink: command execution
  and template renderers, file operations and path joins, SQL/NoSQL query construction,
  deserialization calls, redirects, and proxying logic.
- Interpolation, string-built queries, and `eval`-family calls get priority; wrappers and "safe"
  helpers usually are not — read them.

## 3. Audit authentication and authorization explicitly
- Which endpoints skip the auth middleware? Map every route against its expected role.
- Object-level checks (does this user own this record?) separate from function-level checks —
  IDOR lives in the gap.
- Session/token handling: generation, validation, expiry, and secret storage.

## 4. Secrets, dependencies, and history
- Scan for hardcoded keys, tokens, connection strings, and internal hostnames.
- Match exact dependency versions against known CVEs — direct and transitive; vendored binaries
  deserve the same version check.
- Git history: deleted credentials, old config files, and abandoned branches frequently outlive
  "removal".

## 5. Validate before reporting
- Build/run the target locally (or on an authorized staging copy) and reproduce each suspected bug
  with a minimal PoC — apply the `web-app-testing` oracle discipline; a code smell is a finding
  only when it demonstrably fails.

## 6. Report with code references
- Every finding gets file/line references, the vulnerable pattern, a minimal fix suggestion, and
  the validated impact. Hand off per the `report-writing` skill.

---
*Standard, publicly documented secure-code-review methodology as commonly applied in authorized
engagements and supported by the source-analysis patterns observed in public lab writeups.*
