---
name: mobile-testing
description: Mobile application security testing methodology for authorized engagements — Android (APK analysis, decompilation, Frida hooking, SSL pinning bypass, device and backup analysis) and iOS (IPA analysis, keychain, jailbroken-device testing), aligned to OWASP MASVS, plus backend API testing through the app. Use when the target includes a mobile app, its backend APIs, or mobile infrastructure.
metadata:
  short-description: Android/iOS app testing playbook
  category: mobile
---

# Mobile Application Testing

> Authorized engagements only (see `authorized-use`): test the program's own app, on your own
> devices (rooted Android / jailbroken iOS), with test accounts. Never test other users' devices,
> accounts, or production data.

## 1. Set up an authorized lab
- Rooted Android device/emulator with Frida + objection; jailbroken iOS device; an interception
  proxy; a test account with known state.
- Confirm the app under test matches the released version; note platform versions and
  anti-tampering protections.

## 2. Static analysis
- Decompile: `jadx`/`apktool` (Android: manifest, exported components, deep links, backup and
  cleartext flags), class-dump/plist review (iOS).
- Hunt hardcoded secrets (API keys, tokens, endpoints), weak crypto, and debug/test endpoints left
  in the binary.
- Map exported components and IPC surfaces — exported activities/receivers/services and deep links
  are the client-side attack surface.

## 3. Dynamic analysis
- Proxy all traffic; bypass SSL pinning (Frida/objection hooks) rather than skipping pinned flows.
  Inspect local storage: shared preferences, SQLite databases, keystores, plists, keychain —
  secrets often land in plaintext "temporarily".
- Hook sensitive functions (crypto, token handling, auth decisions) to verify logic, not just
  guesses.
- Test IPC/deep-link abuse with crafted intents/universal links under your control.

## 4. The backend is the real prize
- The mobile API usually carries the business logic: test authorization per `web-app-testing`
  (IDOR, role checks, token handling) through the app's requests.
- Check that server-side checks exist — client-side-only validation is the classic mobile finding.

## 5. MASVS-aligned checks
- Storage (MSTG-STORAGE), crypto (MSTG-CRYPTO), authentication/session (MSTG-AUTH), network
  (MSTG-NETWORK), platform interaction (MSTG-PLATFORM), code quality (MSTG-CODE) — cover each class
  systematically rather than ad-hoc.

## 6. Verify and report
- Reproduce every finding deterministically on the lab device; capture request/response plus app
  version; report per `report-writing` with the client-side impact stated concretely.

---
*Methodology aligned to publicly available OWASP MASVS/MSTG guidance and common public mobile
bug-bounty writeup patterns. No proprietary material reproduced.*
