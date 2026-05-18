# KHAOSS Stack

**K**anban · **H**ermes · **A**ionUI · **O**bsidian · **S**creenpipe · **S**pace Agent

> Local AI command centre. Screen capture + multi-agent orchestration +
> privacy vault. Self-hosted, privacy-first.
> *If-Need-Bee — KHAOSS today, BOHKSS tomorrow.*

---

## Install (double-click)

Download this repo as a zip, extract, then:

| Platform | Double-click this file |
|---|---|
| **Windows** | `installers/KHAOSS-Install-Windows.bat` |
| **macOS** | `installers/KHAOSS-Install-macOS.command` (right-click → Open if blocked) |
| **Linux** | `installers/KHAOSS-Install-Linux.sh` |

Or one-line terminal install:
```bash
# Linux / macOS
curl -fsSL https://raw.githubusercontent.com/heimdallthegatekeeper1/KHAOSS-STACK/main/installers/install.sh | bash
```

**Something went wrong?** → See [INSTALL_HELP.md](INSTALL_HELP.md)

---

## What you get

| | Service | What it does |
|---|---|---|
| K | **Kanban** | Task board — AI agents pick up and complete tasks |
| H | **Hermes v0.14.0** | Multi-provider AI gateway — Codex OAuth, Claude, GPT-5.5 |
| A | **AionUI** | Electron desktop app for multi-agent team sessions |
| O | **Obsidian** | Session memory + weekly learning notes auto-written here |
| S | **Screenpipe** | Screen + audio capture with PII scrubbing |
| S | **Space Agent** | Opt-in autonomous desktop agent |

All runs on localhost. Nothing leaves your machine except what you
explicitly send to an AI provider.

---

## Control Panel — http://localhost:7842

After running `bash ~/launch-stack.sh`:

- **⚡ Launch All** — one-click boot with live terminal progress
- **Debate Mode** — two AI agents argue any topic, judge delivers verdict
- **Session Memory** — End Session writes your daily Obsidian note
- **Screen Assistant** — AI suggestions based on your vault files
- **Proactive Research** — auto-researches new vault content (needs API key)
- **Learning Loop** — weekly skill synthesis from your work patterns
- **Service Controls** — start/stop/restart every service from the panel
- **Privacy Blocklist** — add sensitive words to never send to AI

---

## Prerequisites

Before running the installer, you need:

| Tool | Where to get it |
|---|---|
| Python 3.10+ | https://python.org/downloads |
| Node.js 18+ LTS | https://nodejs.org |
| Git | https://git-scm.com |
| Screenpipe | https://screenpipe.com |
| AionUI | https://github.com/aionui/aionui (extract to ~/Downloads/AIonUi/AionUi-main + npm install) |

The installer **auto-installs**: Hermes, Codex CLI, Python packages.

Full guide: [docs/DEPENDENCIES.md](docs/DEPENDENCIES.md)

---

## Hermes setup (required, 3 commands)

```bash
hermes auth login --provider openai-codex
hermes profile create swarm13 --provider openai-codex --model gpt-5.5
hermes config set api_server.enabled true --profile swarm13
```

---

## AionUI setup (required for agent teams)

```bash
# After extracting AionUI to ~/Downloads/AIonUi/AionUi-main:
cd ~/Downloads/AIonUi/AionUi-main && npm install

# Verify Hermes integration patches
grep -c "LOCAL PATCH" src/common/types/teamTypes.ts
# Must return 1 — see docs/AIONUI_HERMES_LEADER_FIX.md if 0
```

---

## Privacy

Screenpipe scrubs before any AI sees your screen:

**✓ Auto-scrubbed:** API keys (`sk-`, `ghp_`, `AKIA...`), emails,
phone numbers, SSNs, credit cards, file paths, IPs, URL tokens

**✗ May still reach cloud AI:** passwords in browser forms,
private chat windows, context-specific names

**Safe rules:**
1. Disable Screenpipe in panel during sensitive work (one toggle)
2. Add sensitive words to Privacy Blocklist in panel
3. Check `~/vault/workspace/` to see what agents can read

---

## After install

```bash
bash ~/launch-stack.sh          # boot stack
open http://localhost:7842      # macOS
xdg-open http://localhost:7842  # Linux
bash ~/khaoss/scripts/khaoss_tests.sh  # verify
```

---

## Troubleshooting

**Something broken?**
```bash
bash scripts/khaoss_debug.sh
# Paste the output file into Claude, ChatGPT, or any AI
```

See [INSTALL_HELP.md](INSTALL_HELP.md) for:
- Common error fixes with exact commands
- Timeout detection and recovery
- Full reinstall procedure
- How to use Terminal CC / Claude.ai for AI-assisted fixing

---

## Coming: BOHKSS

When Beewid (new multi-chat AI cockpit) reaches Phase E, AionUI is
retired and the stack renames to BOHKSS. The panel, vault, Hermes,
Kanban, Screenpipe and Space Agent all continue unchanged.

---

## Repository
```
installers/    Double-click + terminal installers (Win/Mac/Linux)
privacy/       Privacy pipeline (scrubber, vault, panel server)
scripts/       Launch, test, and debug scripts
configs/       Default config files
docs/          Technical docs and dependency guide
```

*Verified: Kali Linux (X11) · Debian (X11) · 2026-05-18*
