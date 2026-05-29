# USER_MANUAL.md — dotdrop

Pied Piper fork of [LocalSend](https://github.com/localsend/localsend) — cross-platform file drop with planned Pipernet integration. Transfers files and messages between nearby devices over a local network using mDNS discovery and HTTPS, with no internet or external server required. Stage 1 complete (fork, branch, license); Pipernet identity + relay integration is Stage 2+.

---

## What it is

A Flutter app (Android/iOS/macOS/Windows/Linux) for local-network file sharing, plus a Rust/Axum WebSocket signaling server. The fork adds Pied Piper-specific identity and relay layers on top of LocalSend's mDNS + HTTPS + UDP-multicast core.

### Current fork status (as of 2026-05-02)

- Upstream HEAD at fork: `5ccc6dea feat: move target discovery out of external isolate`
- Branch: `pipernet-fork` off upstream `main`
- Stage 2 integrations (Ed25519 keypair, signed manifests, Pipernet relay, $PIPER receipts) are designed but not yet implemented in code

---

## Components

| Component | Language | Location | Purpose |
|---|---|---|---|
| Flutter app | Dart | `app/` | Cross-platform GUI; file discovery, send/receive |
| Rust server | Rust (Axum) | `server/` | WebSocket signaling for WebRTC peer negotiation |
| Core lib | Rust | `core/` (submodule) | Shared LocalSend protocol types |

---

## Build the Flutter app

**Requirements:** Flutter (version pinned in `.fvmrc`), Rust

```bash
# install fvm if needed
dart pub global activate fvm

# use the pinned Flutter version
fvm install
fvm flutter pub get

# run on connected device / simulator
cd app
fvm flutter run

# build release
fvm flutter build apk          # Android APK
fvm flutter build appbundle    # Android Play Store
fvm flutter build ipa          # iOS
fvm flutter build macos        # macOS
fvm flutter build windows      # Windows
fvm flutter build linux        # Linux
```

---

## Build the Rust signaling server

```bash
cd server
cargo build --release
# binary: server/target/release/server
```

### Run the signaling server

```bash
# defaults: 0.0.0.0:3000
./server/target/release/server

# Override
SERVER_IP=127.0.0.1 SERVER_PORT=8080 ./server/target/release/server
```

### Docker (signaling server)

```bash
docker build -t dotdrop-server -f server/Dockerfile .
docker run -p 3000:3000 dotdrop-server
```

---

## Environment variables (signaling server)

| Variable | Default | Purpose |
|---|---|---|
| `SERVER_IP` | `0.0.0.0` | Bind address |
| `SERVER_PORT` | `3000` | WebSocket listen port |
| `MAX_CONNECTIONS_PER_IP` | `10` | Flood guard per IP group |
| `MAX_REQUESTS_PER_IP_PER_HOUR` | `1000` | Rate limit per IP per hour |

---

## Signaling server API

The server exposes a single WebSocket endpoint used by the Flutter app for WebRTC peer negotiation. HTTP REST is not exposed — all communication is over WS.

### WebSocket endpoint

```
GET /v1/ws?d=<base64(PeerRegisterDto)>
Upgrade: websocket
```

`d` is a base64-encoded JSON `PeerRegisterDto`:

```json
{
  "alias":     "My Device",
  "version":   "2.1",
  "deviceModel": "MacBook Pro",
  "deviceType":  "desktop",
  "fingerprint": "<SHA-256 of device TLS cert>",
  "port":      53317,
  "protocol":  "https",
  "download":  true
}
```

Once connected, the server relays typed messages between peers in the same IP group:

| `WsClientMessage` type | Description |
|---|---|
| `register` | Announce presence; server broadcasts to IP-group peers |
| `sdp_offer` | Send WebRTC SDP offer to a specific peer UUID |
| `sdp_answer` | Send WebRTC SDP answer back to offeror |

| `WsServerMessage` type | Description |
|---|---|
| `joined` | A new peer has joined your IP group |
| `left` | A peer has disconnected |
| `sdp` | Relayed SDP offer or answer |

---

## Network requirements

| Traffic | Protocol | Port | Direction |
|---|---|---|---|
| Signaling WS | TCP | 3000 (configurable) | Inbound |
| LocalSend LAN transfer | TCP + UDP | 53317 | Inbound + outbound |
| mDNS discovery | UDP multicast | 5353 | Both |

Disable AP isolation on your router — devices must be able to reach each other on the LAN. On Windows, set the network profile to "Private."

---

## Observing state

```bash
# Signaling server — is it up?
curl -i --http1.1 -H "Connection: Upgrade" -H "Upgrade: websocket" \
  http://localhost:3000/v1/ws?d=<base64dto>
# Expect: 101 Switching Protocols

# Rust process
ps aux | grep server

# Docker
docker ps
docker logs <container-id> --tail 50

# cargo test
cd server && cargo test
```

---

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Device not visible in app | AP isolation enabled on router | Disable AP isolation; for Windows set network to "Private" |
| WS connection refused | Server not running or wrong port | Check `SERVER_PORT`; confirm firewall allows TCP 3000 |
| Flutter build fails | Wrong Flutter version | Run `fvm use` in `app/` to switch to the pinned version |
| `fvm flutter` not found | fvm not on PATH | `export PATH="$PATH":"$HOME/.pub-cache/bin"` |
| Rust build error on `localsend` dep | Submodule not initialized | `git submodule update --init --recursive` |
| Speed is slow | 2.4 GHz congestion or encryption overhead | Switch to 5 GHz band; known Android SAF issue on receive |

---

## Stage 2 planned integration points (not yet implemented)

Per `FORK-NOTES.md`:

1. **Ed25519 keypair per device** — generated on first launch; public key = Pied Piper address (displayed as QR + `piper://` handle)
2. **Signed transfer manifest** — envelope signed before send; receiver stores signed receipt as two-sided proof
3. **Pipernet relay fallback** — PAKE rendezvous for cross-NAT / internet transfers (iroh-based)
4. **$PIPER receipt emission** — signed transfer event sent to Pipernet backend on success; holders accumulate receipts

The integration points (file:line) are to be documented in `PIPERNET-INTEGRATION.md` during Stage 2.

---

## Key files

| Path | Purpose |
|---|---|
| `app/` | Flutter app source |
| `app/.fvmrc` | Pinned Flutter version |
| `server/src/main.rs` | Axum server entry point |
| `server/src/controller/ws_controller.rs` | WebSocket peer signaling |
| `server/src/config/` | State, init, error types |
| `core/` | Shared Rust types (git submodule) |
| `FORK-NOTES.md` | Fork rationale and Stage 2 plan |
| `CONTRIBUTING.md` | Contribution guide |

---

*This is a Pied Piper fork of an Apache 2.0 project. Stage 1 complete. Stage 2 (Pipernet identity + relay) is next. Build agents should consult `FORK-NOTES.md` before modifying the transfer or discovery stack.*
