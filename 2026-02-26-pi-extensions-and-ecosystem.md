# Pi Extensions & Skills Ecosystem — Feb 2026

## Source session
- **Session file:** `~/.pi/agent/sessions/--C--Users-jackc-AppData-Local-Temp--/2026-02-26T12-19-04-071Z_7cf44fff-3583-4c6a-a665-b8954ceee879.jsonl`
- **Session ID:** `7cf44fff-3583-4c6a-a665-b8954ceee879`

---

## Overview

Pi (pi-mono) has ~17K GitHub stars and a healthy extension ecosystem driven by its "build it yourself" philosophy. The community hub is the Shitty Coders Club Discord (2,410 members). OpenClaw (217K stars) is the flagship SDK integration, but the independent adoption story is substantial — people switch from Claude Code for the lean system prompt and better token efficiency.

## What everyone builds (Tier 1 — multiple independent implementations)

### Subagents — the #1 thing
At least 5 independent implementations exist. Mario's own take: "There are many subagent extensions, this one is mine."
- Pi's own example extension (`examples/extensions/subagent/`)
- mjakl/pi-subagent
- richardgill's task-tool (isolated pi subprocesses, parallel/chain/single)
- mitsuhiko's loop extension (iterative dev loops)
- tmustier's ralph-wiggum (long-running agent loops)

### Notifications ("agent is done")
4+ independent implementations: pi-notification-extension, pi-notify-pp, ferologics/pi-notify, joshuadavidthomas's notify (cheap model summary like "Wrote auth.ts" or "Need: which database?").

### Cost/usage tracking
4+ implementations: hjanuschka's cost-tracker, pi-cost-dashboard (mrexodia, full web dashboard), splitrail (cross-agent: Pi + Claude Code + Codex).

### Permission/security gates
Multiple approaches: Pi's own example, prateekmedia's 4-level system, michalvavra's dangerous command blocker, kcosr's toolwatch (SQLite audit logging), rhubarb-pi/safe-git.

### Plan mode
Pi's own example extension, hjanuschka's plan-mode (read-only exploration), Plannotator (visual plan annotation).

## Common patterns (Tier 2)

- **Handoff/context transfer** — extract context → new session instead of compacting
- **Memory/AGENTS.md management** — save instructions to AGENTS.md; community consensus: just use markdown files
- **Checkpoint/rewind** — arpagon/pi-rewind, nicobailon/pi-rewind-hook (all git-based)
- **Code review** — victorarias/shitty-reviewing-agent ("better than Claude/Codex/Greptile/Copilot"), mitsuhiko's review extension
- **Messenger/chat bridges** — tintinweb's pi-messenger-bridge (Telegram/WhatsApp/Slack/Discord)
- **Session decoration** — emoji, colours, powerline footer, ghostty title bar, themes (tokyonight, rose-pine)

## Notable extensions & tools

| Extension | Author | What |
|-----------|--------|------|
| **oh-my-pi** | can1357 | Major fork: LSP, Python tool, TTSR, commit tool |
| **pi_agent_rust** | Dicklesworthstone | Full Rust port — 5x faster, 12x less memory |
| **pi-coding-agent (Emacs)** | dnouri | Full Emacs frontend (on MELPA) |
| **agent-stuff** | mitsuhiko (Armin Ronacher) | review, loop, todos, commit, Sentry, tmux |
| **shitty-extensions** | hjanuschka | 7+ extensions: cost-tracker, handoff, memory-mode, oracle, plan-mode, ultrathink |
| **pi-messenger-bridge** | tintinweb | Bridge Telegram/WhatsApp/Slack/Discord into Pi |
| **pi-canvas** | jyaunches | Interactive TUI canvases rendered inline |
| **pi-ssh-remote** | cv | Redirects all file ops to remote host via SSH |
| **mcporter** | steipete | Wraps MCP servers as CLI tools |
| **nono** | lukehinds | Kernel-enforced sandbox (Landlock/Seatbelt) |

## Prolific community members

| Author | Ships |
|--------|-------|
| **hjanuschka** | shitty-extensions (7+), pi-qmd, pi-entire, pi-ghostty |
| **tintinweb** | messenger-bridge, todo-list, schedule-prompt, VS Code provider |
| **mitsuhiko** (Armin Ronacher) | agent-stuff: review, loop, files, todos, cwd-history, codex-tuning |
| **joshuadavidthomas** | agentkit: answer, beans, dcg, handoff, messages, notify, providers |
| **nicobailon** | pi-interview-tool, powerline-footer, prompt-template-model, rewind-hook |
| **juanibiapina** | extension-settings, tokyonight theme, gob process manager |
| **benvargas** | pi-packages, ancestor-discovery, firecrawl-mcp, exa-mcp |

## Key ecosystem dynamics

1. **"Ask pi to build it"** — the most common advice. The prolific users all have personal extension repos that are things they asked pi to build for them.
2. **Skills are cross-compatible** — pi SKILL.md format works with Claude Code, Codex CLI, Amp, and Droid. Deliberately simple markdown.
3. **No MCP servers** — deliberate philosophy. A few people wrap existing MCP servers via mcporter, but nobody builds native MCP for Pi.
4. **No VS Code integration** — terminal-native is the culture. Only tintinweb's model chat provider bridge exists.
5. **Single maintainer concern** — bus factor = 1 (Mario Zechner). The Rust port and oh-my-pi fork are hedges.

## Notable endorsements

- **Armin Ronacher** (Flask creator): "Pi is written like excellent software"
- **Ewald Benes**: "10x longer token limits for same volume of work" vs Claude Code
- **Nader Dabit**: Wrote the canonical SDK tutorial
- **Raspberry Pi Foundation**: Featured OpenClaw/Pi on their official blog
