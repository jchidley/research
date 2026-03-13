# Remote Pi Access — Five Approaches Compared

## Source session
- **Session file:** `~/.pi/agent/sessions/--C--Users-jackc-git-pi-messenger-bridge--/2026-02-27T19-06-37-787Z_5a6f89f2-43d1-4cc3-a0d5-04b73fd94521.jsonl`
- **Session ID:** `5a6f89f2-43d1-4cc3-a0d5-04b73fd94521`

## Related research
- `~/research/2026-02-26-pi-on-my-devices.md` — device fleet mapping, integration patterns
- `~/research/2026-02-26-pi-extensions-and-ecosystem.md` — extension ecosystem survey
- `~/research/2026-02-27-element-x-to-pi5-bridge.md` — Matrix transport implementation

---

## Context

Five distinct ways to use pi remotely against a Raspberry Pi 5 (always-on, on the home LAN at 10.0.1.230), accessed from Windows laptop or Android phone over WireGuard VPN. All assume WireGuard is the network layer.

The five approaches are not competing alternatives — they serve different situations and can run simultaneously on the same Pi 5.

---

## 1. SSH + tmux (pi runs directly on Pi 5)

WireGuard → SSH to Pi 5 → tmux session → pi interactive TUI.

### Pros
- Agent tool loop is entirely local to the Pi 5. No network cost per tool call — a 10-tool-call agent turn runs at full disk/CPU speed.
- tmux sessions survive SSH disconnects, laptop sleep, reboots. Reconnect with `tmux attach`.
- Zero setup — SSH and tmux already exist.
- Pi 5 has the files, the earpiece endpoint, the messenger-bridge — pi can interact with all of it directly.
- Simple mental model: one machine, one process, one filesystem.

### Cons
- Keystroke and screen-update latency over WireGuard. Fine on home WiFi, noticeable on 4G.
- Needs a proper terminal. Poor experience from a phone (Termux + SSH + on-screen keyboard).
- Synchronous — you sit and watch.

### Latency model
You pay latency on typing and screen updates (the cheap parts). The expensive part — the agent's inner loop of think → tool call → think → tool call — runs locally at full speed. For a typical coding session where the agent does 90% of the work, this is the right trade-off.

### When this fits
Coding sessions from a laptop where the agent does the heavy lifting. You type a prompt every few minutes, it runs many tool calls, you read the result.

---

## 2. pi-ssh (hjanuschka) — pi runs on Windows, tools on Pi 5

Pi runs locally on Windows. The pi-ssh extension (`--ssh jack@10.0.1.230:/path`) redirects all four tools (read, write, edit, bash) plus user `!` commands over SSH to the Pi 5.

**Source:** https://github.com/hjanuschka/pi-ssh

### How it works (from reading the source)
- Persistent SSH shell session with environment preserved between bash commands (not one-shot SSH per call).
- SSH multiplexing (`ControlMaster`/`ControlPersist`) — first connection is slow, subsequent tool calls reuse the socket.
- Path mapping: local `$HOME` → remote `$HOME`. System prompt `cwd` rewritten to reflect remote path.
- Streaming output via start/end markers — bash output appears incrementally in the TUI, not buffered until completion.
- Graceful fallback: if `--ssh` not passed, all tools are local. One extension, both modes.

### Pros
- Local TUI — zero keystroke latency. Full responsiveness while typing, scrolling, reading.
- API keys and billing stay on Windows. No need to configure them on Pi 5.
- Local session files — easy to browse, search, fork from Windows.
- All local extensions work natively.
- Can use Windows-side tools alongside (VS Code, browser) without context-switching.

### Cons
- Every tool call pays a WireGuard + SSH round-trip. A 10-tool-call agent turn accumulates that serially through the SSH command queue.
- Split brain — pi process on Windows, filesystem on Pi 5. WireGuard drop mid-session = stalled command queue.
- Windows machine must stay awake. No persistence if laptop sleeps (contrast tmux).
- Can't directly interact with Pi 5 services (earpiece endpoint, running processes).
- Duplicates API key management if keys are also needed on Pi 5 for messenger-bridge and earpiece anyway.

### Latency model
Opposite of SSH+tmux. You pay zero latency on typing and reading (local TUI), but every tool call in the agent loop crosses the network. For a turn with 10 tool calls at 50ms RTT each, that's 500ms of pure network overhead the agent wouldn't have with SSH+tmux.

### When this fits
When you want to stay in your Windows workflow — local terminal, local session management, local extensions. When switching between local and remote projects in the same pi instance. When the Pi 5 doesn't have everything you need locally (though bash is remote, so this is limited).

---

## 3. pi-messenger-bridge (Matrix / Element X)

Pi runs 24/7 on Pi 5 with the messenger-bridge extension. Chat via Element X on phone or Element Web on laptop.

**Source:** https://github.com/jchidley/pi-messenger-bridge (fork with Matrix transport)

### Pros
- Use from anywhere with a chat app. Phone in pocket, quick message, read response later.
- Async by nature — fire and forget, notification when done.
- Multi-device — Element X on phone, Element Web on laptop, same conversation.
- Already built, deployed, E2EE working.
- Works without WireGuard — Matrix protocol traverses the internet via matrix.org.
- Lowest barrier to interaction — no terminal, no SSH, just a chat app.

### Cons
- Lossy interface — no TUI, no tool approval, no streaming, no colour.
- Complex multi-file coding is awkward in chat bubbles.
- Message size limits for large code output.
- Matrix infrastructure dependency (matrix.org or self-hosted Synapse/Conduit).
- Security surface — bot account, access tokens, crypto store.
- You trust pi to do the right thing with no interactive approval.

### When this fits
Quick questions on the go. "What's the status of X?" "Read the last earpiece session and summarise it." "Restart the analysis endpoint." Short, self-contained tasks where you don't need to supervise. The only option that works well from a phone while walking.

---

## 4. Android app + Pi 5 backend (generalised earpiece model)

A custom Android app sends requests to a Pi 5 HTTP/WebSocket server that wraps pi via SDK (`createAgentSession()`) or RPC (`pi --mode rpc`).

### The pattern earpiece already proves

Earpiece's architecture is: Android app → HTTP POST `{system, prompt}` → Pi 5 endpoint → Claude API → text response. The Pi 5 endpoint is currently a thin Claude proxy in `AnalysisClient.kt`:

```kotlin
val json = JSONObject().apply {
    put("system", systemPrompt)
    put("prompt", "Recent conversation:\n\n$lines")
}
// POST to http://10.0.1.230:8080/analyse
```

This is a specific instance of a general pattern: **any Android app can be a frontend to pi running on the Pi 5**. Replace the thin proxy with pi SDK or RPC and you get the full agent — tools, file access, multi-turn, extensions, skills — behind the same HTTP pattern.

### How the generalised version would work

**Option A — Pi SDK (~50 lines of Node.js on Pi 5):**
```typescript
const { session } = await createAgentSession({
  sessionManager: SessionManager.create("/home/jack/pi-sessions"),
  authStorage: AuthStorage.create(),
  modelRegistry: new ModelRegistry(authStorage),
});
// HTTP/WebSocket endpoint serves session.prompt() and streams events
```

**Option B — Pi RPC (no code, just process management):**
Spawn `pi --mode rpc --no-session`, pipe JSON stdin/stdout. The RPC protocol supports prompt, steer, follow-up, abort, model switching, compaction, session management — everything. Any language that can manage a subprocess can drive it.

### Pros
- Custom UI — whatever Android can render. Voice I/O, tool approval dialogs, streaming text, images.
- Structured responses — JSON events, not chat messages. Parse tool calls, code blocks, errors programmatically.
- Full session and model control via RPC command set.
- Multiple frontends can share the same pi backend simultaneously.
- Can integrate directly with earpiece — same backend, shared context.
- Offline-queueable — app can buffer requests, retry when WireGuard reconnects.

### Cons
- Requires building both sides: HTTP/WebSocket server on Pi 5 + Android client.
- Most engineering effort of all approaches.
- Another thing to maintain — server process, app updates, API versioning.
- RPC protocol is comprehensive but complex (extension UI sub-protocol, streaming events, etc.).

### When this fits
When you want a purpose-built mobile experience beyond chat bubbles. A "pi remote" app with proper streaming, tool approval, session switching. Or specialised single-purpose apps — earpiece is one; a "daily briefing" app or "home automation via pi" app could be others. The value is in the custom UX, not just access.

---

## 5. pi_agent_rust (Dicklesworthstone)

Complete Rust rewrite of pi. Same CLI, same RPC mode, same JSONL session format. Single static binary.

**Source:** https://github.com/Dicklesworthstone/pi_agent_rust

### Performance characteristics

| Metric | TypeScript pi | Rust pi |
|--------|--------------|---------|
| Startup | ~1s | <100ms |
| Memory (idle) | ~200MB | <50MB |
| Memory (1M token session) | ~820MB | ~68MB |
| Session resume (1M tokens) | ~120ms | ~18ms |
| Binary | Node.js runtime + npm | Single static binary <22MB |

### Pros
- 4x less memory — meaningful on Pi 5 running multiple services 24/7 (pi + earpiece endpoint + messenger-bridge + other services).
- <100ms startup makes `pi -p "query"` genuinely instant for scripting and cron.
- Single binary, no Node.js runtime dependency on the Pi 5.
- Same RPC mode and session format — could serve as backend for any of the other approaches.
- Faster session resume on large histories.

### Cons
- Extension compatibility is work-in-progress — runs JS extensions via QuickJS, not Node.js. pi-messenger-bridge and pi-ssh may need adaptation.
- Single maintainer who explicitly doesn't accept contributions. Bus factor = 1.
- Separate project, separate release cadence from upstream pi.
- Needs Rust toolchain to build on aarch64 (Pi 5).
- Smaller user base, less battle-tested.

### When this fits
As the always-on pi daemon on the Pi 5 — the headless backend serving messenger-bridge, RPC endpoint, and cron jobs — where memory footprint and startup time matter. Less relevant for interactive sessions where you don't notice the difference. Depends on extension compatibility maturing.

---

## How they compose

These are not mutually exclusive. Several can run simultaneously on the same Pi 5.

**Interactive sessions (pick one at a time):**
- SSH+tmux and pi-ssh are different ways to get an interactive TUI. You'd use one or the other depending on whether you want the agent loop local to Pi 5 (SSH+tmux) or the TUI local to Windows (pi-ssh). Not both at once.

**Always-on services (can all run concurrently):**
- Messenger-bridge runs as an extension inside a pi instance on the Pi 5. That instance could be the one you SSH into via tmux, or a separate always-on instance.
- Android app + SDK/RPC would be a separate server process, potentially sharing session files.
- pi_agent_rust could replace TypeScript pi in any of the above roles if extension compat allows.

**Shared vs separate pi instances:**
One pi instance serving multiple frontends (SSH + messenger-bridge + RPC for Android app) means shared context and session history. Separate instances mean isolation and independent lifecycle. Both are valid; the choice depends on whether shared context helps or hinders.

**The Pi 5 as hub:**
All approaches except pi-ssh converge on pi running on the Pi 5. The Pi 5 is the natural hub — it has the files, runs the earpiece analysis endpoint, has the API keys, is always on. The question is just which frontend you're using at any given moment.

---

## Open questions

- **Self-hosted Matrix on Pi 5?** Conduit or Synapse on the Pi 5 would remove the matrix.org dependency for messenger-bridge. Traffic would stay entirely on WireGuard. Worth investigating for LAN-only use.
- **pi_agent_rust extension compat timeline?** The QuickJS runtime is functional but may not handle all Node.js APIs that messenger-bridge and pi-ssh depend on. Worth testing periodically.
- **Shared pi instance vs dedicated?** If messenger-bridge runs inside the same pi instance you SSH into, a remote user's message could interrupt your interactive session. Separate instances avoid this but fragment session history.
- **Android RPC client library?** The pi RPC protocol is well-documented. A thin Kotlin client library wrapping the JSON protocol would make the generalised earpiece model much cheaper to build. Nothing like this exists yet.
