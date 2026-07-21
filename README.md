<div align="center">

# Pied Piper Drop &nbsp;·&nbsp; `dotdrop`

**AirDrop for everyone. Send files device-to-device — no cloud, no account, no sign-in.**
Cross-platform (Mac · Windows · Linux · iOS · Android). Works on a LAN with no internet at all.

[![License: Apache 2.0](https://img.shields.io/badge/license-Apache_2.0-green.svg)](LICENSE)
[![built on LocalSend](https://img.shields.io/badge/built%20on-LocalSend-2563eb.svg)](https://github.com/localsend/localsend)
[![Pied Piper](https://img.shields.io/badge/family-Pied%20Piper-00ff41.svg)](https://github.com/dot-protocol/piedpiper)

[What it is](#what-it-is) ·
[Build &amp; run](#build--run) ·
[DOT-native roadmap](#the-dot-native-roadmap) ·
[Credits](#credits)

</div>

---

**Pied Piper Drop** is the file-transfer member of the [Pied Piper](https://github.com/dot-protocol/piedpiper) family — the internet you own, not rent. Pied Piper chat carries your *messages* peer-to-peer; Pied Piper Drop carries your *files* the same way: straight from one device to another, encrypted in transit, with no server in the middle holding your data and no account to create.

## What it is

- **Device-to-device transfer** — files go directly between devices over your local network (mDNS discovery + HTTPS). Nothing is uploaded to a cloud.
- **No account, no sign-in, no cloud** — open it, pick a file, pick a device, send. That's the whole flow.
- **Truly cross-platform** — Mac, Windows, Linux, iOS, Android. An AirDrop that isn't locked to one vendor.
- **Offline-first** — on a LAN it needs zero internet.

## Build & run

Pied Piper Drop is built on the proven [LocalSend](https://github.com/localsend/localsend) engine (Flutter). Build from source:

```bash
git clone https://github.com/dot-protocol/dotdrop
cd dotdrop/app
flutter pub get
flutter run          # or: flutter build <macos|windows|linux|apk|ipa>
```

> Prebuilt Pied Piper releases are on the way. Until then, build from source above.

## The DOT-native roadmap

Drop starts as a clean LocalSend fork and grows into a **DOT-native** transfer tool — the same "you own it" thesis as the rest of Pied Piper. What's landing (tracked in [`FORK-NOTES.md`](FORK-NOTES.md)):

- [ ] **Ed25519 identity per device** — your device's public key *is* its Pied Piper address (QR + `piper://` handle). Not an account — a real cryptographic identity.
- [ ] **Signed transfer receipts** — the manifest is signed before send; the receiver keeps a signed receipt. Two-sided, tamper-evident proof a transfer happened.
- [ ] **Relay fallback across NAT** — a Pied Piper rendezvous relay (PAKE) so transfers work across networks and continents, not just the LAN.
- [ ] **Signed transfer events** — optional, for holders who want an on-chain receipt.

*These are the plan, not shipped yet — the device-to-device file core above works today.*

## Credits

Pied Piper Drop is a fork of **[LocalSend](https://github.com/localsend/localsend)** by Tien Do Nam and contributors (Apache-2.0). Full attribution in [`NOTICE.md`](NOTICE.md); the original license is preserved in [`LICENSE`](LICENSE). Standing on their shoulders — the millions who already know "the AirDrop alternative" didn't put it down, and neither did we.

## License

Apache License 2.0 (inherited from LocalSend, patent grant included) — see [`LICENSE`](LICENSE).
