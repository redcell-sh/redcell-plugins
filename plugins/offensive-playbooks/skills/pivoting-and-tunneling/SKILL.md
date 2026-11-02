---
name: pivoting-and-tunneling
description: Network pivoting and tunneling playbook for authorized engagements — mapping what a foothold can reach, building tunnels (SSH local/remote/dynamic, SOCKS proxies, chisel/socat), port-forwarding to localhost-bound and internal services, and chaining pivots through multi-host networks. Use whenever a foothold needs to reach hosts or ports that are not directly accessible.
metadata:
  short-description: Foothold → internal network reach
  category: exploitation
---

# Pivoting and Tunneling

> Authorized engagements only (see `authorized-use`). Pivoting is where mixed-OS networks are
> actually conquered: web host → internal host → file server → domain controller, one tunnel at a
> time. Every tunnel must be in scope and logged.

## 1. Map what the foothold can reach first
- `ip route` / `ifconfig`, ARP tables, and internal DNS — establish which segments and hostnames the
  foothold can resolve and route to before building anything.
- Scan internal ranges **one segment at a time**, targeted (the services you care about: SMB, RDP,
  WinRM, web, databases). Full scans through tunnels are slow — triage first.
- Read shares, login scripts, and config files found on the foothold host — they frequently contain
  the credentials and topology for the next hop.

## 2. Choose the tunnel to fit the job
- **Forward a single port back**: `ssh -L` (from an SSH-capable foothold) or `socat` relays; on
  Windows footholds, `netsh interface portproxy`.
- **Reverse a port out**: when the target initiates (SSH `-R`), or a callback shell needs a listener
  reachable from inside.
- **Full SOCKS pivot**: `ssh -D` or `chisel`/`ligolo`-style reverse SOCKS; route your tooling through
  `proxychains` with a strict exclude list so you don't leak traffic outside the tunnel.
- Prefer existing protocols (SSH, SMB where authorized) before dropping new binaries; if you must
  upload tools, record where they landed and remove them.

## 3. Reach localhost-bound and internal services
- Admin panels, APIs, and databases bound to `127.0.0.1` on a foothold host are often the weak
  point; forward them locally and attack with the full client-side toolkit.
- Websocket and internal-HTTP services: enumerate them through the tunnel exactly as external ones
  (see `web-foothold-patterns`).

## 4. Chain pivots deliberately
- Multi-host chains (A → B → C) require nested tunnels or a routed SOCKS setup; label each hop in
  notes (`tunnel-1: hacky → 10.10.10.20:445`), and validate each new credential against the *new*
  segment before building the next hop.
- Domain resources reachable only from a member host (shares readable only from the file server,
  DC services firewalled to members) are the classic multi-hop payoff.

## 5. Operate and clean up
- Keep tunnel process inventory (ports, PIDs, hosts) in the engagement log; kill every listener at
  the end and verify nothing is left listening on the target.
- Re-verify each pivot still reproduces if the engagement restarts — tunnels are part of the chain
  you must be able to document.

---
*Patterns distilled clean-room from public community lab writeups (0xdf hacks stuff and similar
public walkthrough material). No writeup text or flags reproduced.*
