# Element X → Pi 5 Bridge via pi-messenger-bridge — Feb 2026

## Source sessions
- **Research session:** `~/.pi/agent/sessions/--C--Users-jackc-AppData-Local-Temp--/2026-02-26T12-19-04-071Z_7cf44fff-3583-4c6a-a665-b8954ceee879.jsonl`
- **Implementation session:** `~/.pi/agent/sessions/--C--Users-jackc-AppData-Local-Temp--/2026-02-26T19-36-38-066Z_0fdfcec7-6410-4577-8e22-452d7a8f6fa7.jsonl`

---

## What was built

Added Matrix transport support to tintinweb's pi-messenger-bridge extension, enabling Element X on phone → matrix.org → pi coding agent on Pi 5.

## Fork & PR

- **Fork:** https://github.com/jchidley/pi-messenger-bridge
- **Branch:** `add-matrix-transport`
- **PR:** https://github.com/tintinweb/pi-messenger-bridge/pull/3
- **Local clone (Windows):** `~/git/pi-messenger-bridge`
- **Local clone (Pi 5):** `~/pi-messenger-bridge`

## Files changed

| File | Change |
|------|--------|
| `src/transports/matrix.ts` | New — `MatrixProvider` implementing `ITransportProvider` (~240 lines) |
| `src/index.ts` | Import, env vars, auto-connect, configure command, help text |
| `src/types.ts` | Added `matrix` to `MsgBridgeConfig` |
| `README.md` | Matrix setup docs, config example, env vars |
| `package.json` | Added `matrix-bot-sdk` dep, updated description + keywords |

## Matrix transport features

- Uses `matrix-bot-sdk` (npm)
- Auto-joins rooms on invite
- Same 6-digit OTP challenge auth as all other transports
- Rich HTML formatting (code blocks, bold, italic, links)
- Typing indicators
- Group chat with bot mention detection
- Persistent sync state (`~/.pi/msg-bridge-matrix-store.json`)
- Configure via `/msg-bridge configure matrix <homeserver-url> <access-token>` or env vars (`PI_MATRIX_HOMESERVER`, `PI_MATRIX_ACCESS_TOKEN`)

## Setup completed on Pi 5

1. Cloned fork on Pi 5: `git clone -b add-matrix-transport https://github.com/jchidley/pi-messenger-bridge.git`
2. Built: `npm install && npm run build`
3. Installed: `pi install /home/jack/pi-messenger-bridge`
4. Configured: `/msg-bridge configure matrix https://matrix.org <access-token>`
5. Status shows: `✅ Matrix configured and connected` with `💬 [mat]` indicator

## Matrix account setup

- Bot account on matrix.org (reused from prior OpenClaw setup)
- Logged into Element Web to get access token (Settings → Help & About → Access Token)
- Recovery key saved
- Ghost device `agcbUUgUkR` from old OpenClaw session — cannot be deleted due to matrix.org OIDC migration breaking the `DELETE /devices/{id}` API (known issue: https://github.com/cinnyapp/cinny/issues/2376). Harmless, cosmetic only.

## Issues resolved (2026-02-27)

- **M_FORBIDDEN errors on startup:** Initial sync replayed events from rooms the bot had left (Matrix.org Official Account room with `users_default: -10`). Fixed by adding `getJoinedRooms()` guard in `handleMessage` — commit `c4992de`.
- **Messages not received:** Element X encrypts rooms by default (Megolm E2EE). Bot uses `matrix-bot-sdk` which can't decrypt. Created new unencrypted room `pi5data-unencrypted` — works perfectly.
- **Bot left Matrix.org Official Account room** (`!VCKDdBjITIStzTDvaO:matrix.org`) — was causing M_FORBIDDEN errors trying to send challenge codes to `@server:matrix.org`.

## Architecture (working)

```
Phone (Element X)
    │
    │ Matrix protocol (E2EE not used — matrix-bot-sdk limitation)
    ▼
matrix.org homeserver
    │
    │ /sync polling
    ▼
Pi 5 (pi-messenger-bridge extension inside pi)
    │
    │ pi.sendUserMessage() / turn_end events
    ▼
Pi coding agent (interactive session)
```

## E2EE — WORKING (2026-02-27)

E2EE now works using `matrix-bot-sdk` + `RustSdkCryptoStorageProvider` (backed by `@matrix-org/matrix-sdk-crypto-nodejs`, native Rust with SQLite on disk). Crypto state persists at `~/.pi/msg-bridge-matrix-crypto/`.

**Setup required for E2EE:**
1. Log in via password to get a fresh device: `curl -X POST https://matrix.org/_matrix/client/v3/login -d '{"type":"m.login.password",...}'`
2. Update `~/.pi/msg-bridge.json` with new `accessToken`
3. Clear old crypto store: `rm -rf ~/.pi/msg-bridge-matrix-crypto ~/.pi/msg-bridge-matrix-store.json`
4. Start pi — bot registers new device with crypto keys
5. Verify the device from Element Web (log in as bot account, approve "was this you?" prompt)
6. Send new messages from Element X — they decrypt automatically

**Note:** `matrix-js-sdk` was also investigated (branch `add-matrix-js-sdk`) but its E2EE in Node.js uses ephemeral in-memory crypto (no IndexedDB), which doesn't work for bots. See `matrix-org/matrix-js-sdk#4769`. OpenClaw uses the same `matrix-bot-sdk` + `RustSdkCryptoStorageProvider` approach.

## Limitations

- **matrix.org device deletion broken** — OIDC migration means old compatibility sessions can't be cleaned up via API. Only account.matrix.org can manage sessions, but it doesn't show pre-OIDC ghost devices.
- **`htmlencode` deprecation warning** — `matrix-bot-sdk` depends on `htmlencode` (2013) which uses `util._extend`. Suppress with `NODE_NO_WARNINGS=1 pi`.

## Next steps

- ~~Fix room power level for bot~~ → resolved: left problematic room, created unencrypted room
- ~~Test full round-trip~~ → ✅ working: Element X → matrix.org → pi-messenger-bridge → Pi 5 agent → response back
- ~~Add E2EE support~~ → ✅ working: `RustSdkCryptoStorageProvider` with SQLite, verified device `fS5nKiwzGU`
- Consider self-hosted Conduit/Synapse on Pi 5 for LAN-only use over WireGuard
