---
name: database-attack-paths
description: Database attack-path playbook for authorized engagements — MSSQL enumeration and abuse (xp_cmdshell, linked servers, impersonation), MySQL, PostgreSQL, Redis and NoSQL paths from credentials to shells and lateral movement, plus credential harvesting inside databases. Use whenever database credentials, database ports, or database-backed applications are in play.
metadata:
  short-description: Database cred → shell → lateral playbook
  category: exploitation
---

# Database Attack Paths

> Authorized engagements only (see `authorized-use`). Databases are both foothold targets and
> credential vaults — treat any database credential as a probable shell and a probable pivot.

## 1. Fingerprint and collect credentials first
- Identify engine + exact version from banners and error pages; every technique below is
  version-gated.
- Harvest credentials before brute force: app configs, connection strings, memory dumps, backups,
  and code repositories routinely contain working DB credentials.

## 2. MSSQL (the engagement workhorse)
- Enumerate with valid creds (netexec-style mssql modules): logins, roles, links, and whether
  `sysadmin`.
- Enable `xp_cmdshell` when you own the server; if disabled, `sp_configure` can re-enable it when
  authorized.
- **Linked servers / trusted links**: chain them — a "trusted" link frequently executes in the
  remote server's context, and links-of-links are a classic silent path to a domain-privileged
  service account on another host.
- **Impersonation**: `EXECUTE AS` rights on users/procs escalate inside the engine even without
  sysadmin.
- UNC-path tricks (backup/restore to your share) capture authentication handshakes from the DB
  service account — often a Kerberoastable machine/service principal.
- Service account hunting: the MSSQL service runs as a domain account with SPNs → Kerberoast
  candidate once you can authenticate.

## 3. MySQL / PostgreSQL
- MySQL: `LOAD_FILE` reads, `INTO OUTFILE` writes to web roots (webshell) or plugin dirs (UDF
  shell), password-table extraction — minimize, document.
- PostgreSQL: `COPY ... TO/FROM PROGRAM` (superuser RCE), `pg_read_file`, large-object writes;
  extension and role abuse when non-superuser.

## 4. Redis and NoSQL
- Redis: `CONFIG SET` write primitives (webshell/authorized_keys/cron) and module loading on
  exposed instances.
- MongoDB/NoSQL: authentication state first, then injection patterns against the app; data
  extraction must be minimized and logged.

## 5. Inside the database
- User tables, connection strings, stored-proc definitions, and job histories hold the *next*
  credential far more often than the DB itself is the exploit — enumerate deliberately.

## 6. Verify and document
- Record engine, version, credential source, and the exact statements used; database actions are
  logged (audit trails) — keep every action reproducible and defensible.

---
*Techniques reflect standard, publicly documented database attack paths commonly exercised in
authorized labs and community writeup material. No vendor-confidential or target-specific content.*
