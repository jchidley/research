# Pi Coding Agent Across My Device Fleet — Feb 2026

## Source session
- **Session file:** `~/.pi/agent/sessions/--C--Users-jackc-AppData-Local-Temp--/2026-02-26T12-19-04-071Z_7cf44fff-3583-4c6a-a665-b8954ceee879.jsonl`
- **Session ID:** `7cf44fff-3583-4c6a-a665-b8954ceee879`

---

## Device → Pi mode mapping

| Device | Runs Pi? | Best mode | Role |
|--------|----------|-----------|------|
| **Windows (MSYS2/Git Bash)** | ✅ native | Interactive TUI | Primary workstation |
| **WSL2 Debian** | ✅ native | Interactive / SDK / RPC | Dev environment, builds |
| **WSL2 Alpine** | ✅ native | Print/JSON, SDK | Lightweight scripting |
| **LineageOS / GrapheneOS** | ✅ via Termux+proot | Interactive (Ruuh-style) | Mobile access |
| **Raspberry Pi 5** | ✅ native (Node 20+) | Interactive / RPC / SDK | Always-on agent host |
| **Raspberry Pi 4** | ✅ native | Same as Pi 5 | Secondary host |
| **Raspberry Pi Zero 2** | ✅ but tight (~512MB) | Print/JSON (`pi -p`) | Lightweight relay, single-shot |
| **Pi Pico / Pico 2 W** | ❌ no OS | Thin client target | Hardware controlled BY Pi |
| **ESP32** | ❌ no OS | Thin client target | Hardware controlled BY Pi |

## Integration patterns that exist

### 1. pi-messenger-bridge (tintinweb) — ✅ WORKING with Matrix
- Bridges Telegram, WhatsApp, Slack, Discord, **Matrix** INTO a running Pi instance
- 6-digit OTP auth for remote users
- **Deployed:** Pi 5 runs pi, Element X on phone connects via matrix.org
- Matrix transport added via fork: https://github.com/jchidley/pi-messenger-bridge (PR #3 to upstream)
- Install from fork: `pi install /home/jack/pi-messenger-bridge`
- See `~/research/2026-02-27-element-x-to-pi5-bridge.md` for full details

### 2. Pi RPC mode (`pi --rpc`)
- JSON protocol over stdin/stdout, headless, no TUI
- Key building block for bridging — any language can spawn `pi --rpc` and send/receive JSON
- mmeyer/pi-agent-example: full HTTP+WebSocket gateway routing Telegram/Discord/WebChat into isolated pi RPC processes

### 3. Pi SDK (`createAgentSession`)
- Embed Pi in Node.js apps — ~50 lines per Nader Dabit's tutorial
- This is how OpenClaw does it

### 4. Pi print mode (`pi -p "query"`)
- Single-shot, no session, stdout. Perfect for scripting/cron on constrained devices (Pi Zero 2)

### 5. SSH remote (cv/pi-ssh-remote)
- Pi runs locally on Windows, all file operations redirected to remote host via SSH
- Good for Pi 4/Zero 2 as satellites

## Android specifics

**Ruuh** (github.com/perminder-klair/ruuh) — the serious project:
- One-command setup in Termux: installs Ubuntu via proot-distro, then pi-coding-agent on top
- Three Termux API skill files give Pi access to Android hardware:
  - `termux-device` — battery, brightness, torch, vibrate, GPS, WiFi, clipboard, notifications
  - `termux-comms` — SMS, contacts, call log, camera, microphone, TTS, media, calendar
  - `termux-system` — job scheduling, infrared, USB, NFC
- Supports local models via Ollama or cloud APIs
- Pi itself has official Termux support in the README

**pi-mobile** — Android remote client (less documented; uses WireGuard for connectivity)

## Microcontroller control (Pico 2 W / ESP32)

**pico-claw-agent** (github.com/yksanjo/pico-claw-agent) — thin-client architecture:
```
Pi agent (reasoning) ←—JSON over USB Serial—→ Pico (MicroPython firmware)
```
- JSON serial command protocol for GPIO read/write, PWM, ADC, I2C, SPI
- AI reasons → sends structured commands → Pico executes → returns status
- Works with any LLM backend including Pi's SDK

**For fully offline**: PicoLM (1B param LLM in ~2,500 lines of C) + PicoClaw. Runs on Pi Zero 2 at ~2 tok/s, 45MB RAM. Impractical but exists.

## Realistic architecture for my fleet

```
┌──────────────────────────────────────────────────────┐
│           Windows (MSYS2 / WSL2 Debian)              │
│  Pi Interactive — primary coding workstation         │
│  Skills: pi-skills, android-adb, pico-firmware, etc  │
└──────────────┬───────────────────────────────────────┘
               │ SSH / WireGuard
               ▼
┌──────────────────────────────────────┐
│         Raspberry Pi 5               │
│  Pi running 24/7 (RPC or SDK mode)  │
│  pi-messenger-bridge → Matrix/Element│──→ phones
│  USB serial to Pico 2 W             │    (LineageOS/GrapheneOS)
│  SSH to Pi 4 / Pi Zero 2            │
└──────┬───────────────┬───────────────┘
       │               │
       ▼               ▼
┌──────────┐    ┌──────────────┐
│ Pico 2 W │    │ Pi 4 / Zero 2│
│ ESP32    │    │ Pi print mode│
│ (serial/ │    │ satellite    │
│  WiFi)   │    │ tasks        │
└──────────┘    └──────────────┘
```

## What I'd need to build

1. **Pi skill for Pico serial control** — pico-claw-agent has the right protocol but nobody's wrapped it as a SKILL.md. Natural extension of existing `pico-firmware` skill.
2. **Pi skill for ESP32** — same gap, MQTT or HTTP commands wrapped as a skill.
3. ~~**Signal bridge**~~ — **Solved via Matrix.** Element X works on GrapheneOS. pi-messenger-bridge now has Matrix transport.
4. **Session sync across devices** — Pi sessions are JSONL files. No cross-device sync exists. Options: git repo or NFS mount.
5. **WireGuard-aware routing** — no extension detects which device you're on and routes to the right Pi instance.

## What I could grab today

| Need | Tool | Install |
|------|------|---------|
| Message Pi from phone | pi-messenger-bridge (Matrix) | ✅ Done — fork installed on Pi 5 |
| Run Pi on Android | Ruuh | `curl ...setup.sh \| bash` in Termux |
| Control remote Pi host | pi-ssh-remote | `pi install git:github.com/cv/pi-ssh-remote` |
| Notification when done | notify extensions (4+) | Single .ts file |
| Pico serial control | pico-claw-agent | `pip install pyserial` + firmware flash |
