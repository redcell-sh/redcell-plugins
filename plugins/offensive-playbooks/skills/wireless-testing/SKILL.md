---
name: wireless-testing
description: Wireless network testing methodology for authorized environments — 802.11 reconnaissance, WPA/WPA2 handshake capture and offline cracking, WPS and enterprise EAP testing, rogue access point (evil twin) testing strictly within scope, and guest-network segmentation findings. Use when an engagement explicitly includes authorized WiFi networks.
metadata:
  short-description: Authorized WiFi testing playbook
  category: exploitation
---

# Wireless Testing

> Wireless is shared airspace: test only the SSIDs/BSSIDs explicitly named in scope, from the
> authorized location, in the agreed window. Never deauth networks, clients, or channels that are
> not in scope. Wireless laws vary by country — confirm local legality before transmitting.

## 1. Recon (passive first)
- Monitor mode with channel hopping: enumerate SSIDs, BSSIDs, channels, encryption types, and
  client associations; use signal strength to confirm you are testing the in-scope network.
- Map hidden SSIDs, rogue APs already present, and adjacent networks — **note but do not test**
  out-of-scope networks.

## 2. WPA2-PSK testing
- Capture the 4-way handshake for an in-scope client (deauth only that client, minimally, within
  policy) or pull PMKID from the AP where supported.
- Crack offline per the `password-cracking` playbook: targeted dictionaries (organization names,
  venues, room numbers) before brute force. A cracked PSK is a finding — the next step is the
  *network* test, not more cracking.

## 3. WPS and enterprise networks
- WPS: pixie-dust style attacks only where enabled and in scope; otherwise report WPS enablement
  as a finding.
- WPA-Enterprise: enumerate EAP methods; credential attacks (EAP bypass, cert misuse) only within
  explicit rules; certificate validation failures are findings in themselves.

## 4. Rogue AP / evil twin (explicit authorization only)
- Only with written authorization covering the SSID, location, and duration; use an isolated SSID
  and program-owned test credentials for any portal.
- Segment the test AP away from production; capture only what the policy explicitly permits.

## 5. Once on the network
- A WiFi foothold is an internal-network position: run the `host-attack-workflow` against in-scope
  hosts only, and treat guest/corporate segmentation failures (what the guest VLAN can reach) as
  findings — often the most valuable result of a wireless engagement.

## 6. Document
- Record adapters, captures, timestamps, and channel/SSID/BSSID evidence; wireless findings hinge
  on precise RF documentation. Clean up any APs you deployed.

---
*Standard, publicly documented wireless assessment methodology for authorized engagements.*
