# KHAOSS Stack v1.11

A local, multi-agent cowork stack: an AionUi GUI front-end, Hermes-harnessed
agent teams, a three-tier AI router, a control panel, and a Space Agent bridge.
Built to let you spin up a **team of Hermes agents**, give them a goal, and go —
with every agent visible and individually engageable in the GUI, and without the
agents stepping on each other.

**Machine of record:** Cornholio · Kali Linux · X11
**Status:** v1.11 — Hermes team display working via the panel "Build Hermes Team" button.

---

## What this gives you

- **Hermes agent teams that render in AionUi.** A team has a Hermes **leader**
  plus Hermes **teammates**. Each teammate appears as its own card in the AionUi
  panel and can be clicked and talked to directly. Each runs in its own isolated
  session, so a full fleet coexists without GUI conflicts.
- **One-click team building.** The control panel's **Build Hermes Team** button
  provisions a team's teammates for you (see "Building a team" below).
- **Three-tier AI routing** — local Ollama / private / cloud, switchable per need.
- **Space Agent bridge** and **Beewid autocode** bundled.

### Important: what "Hermes agent" means here

The teammates are **Hermes-harnessed agents** (`agentType: hermes`). They run the
Hermes ACP adapter and perform **Hermes skill-building and memory on the backend**.
The *model* behind the harness is configurable — in this build it is
`openai-codex:gpt-5.5`, cloned from the leader — but the agent itself is Hermes,
not a raw Codex agent. "Codex" in the logs refers to the **model provider**, not
the agent type. If you later point a teammate's profile at a different provider
(Anthropic, a local model, etc.), it is still a Hermes agent; only the model
behind it changes.

---

## Install

The stack ships as a double-click installer. The goal is **minimal to no manual
work** beyond the install, with autocode handling environment variance and Claude
Code (CC) available as a fallback for edge cases.

1. **Double-click the installer** (or run the bundled install script). It places
   the stack, applies the required source patches (see "Patches & durability"),
   and builds AionUi.
2. **Launch the stack:** `~/launch-stack.sh`
3. If anything fails to start, run the **autocode** repair flow; for stubborn
   cases, hand the error to **Claude Code**, which can apply the documented
   patches directly.

> **Patches & durability.** Several fixes live in the AionUi source tree and are
> wiped by `npm install`; one lives in the Hermes adapter and is wiped by
> `hermes update`. The installer applies all of them, and a bundled patcher
> re-applies them after any reinstall/update. **If team agents stop appearing
> after an AionUi rebuild or a `hermes update`, re-run the patcher** (see
> "Patches & durability").

---

## Building a team (the main workflow)

### The one-step goal (and where we are)

The intended flow is: create a team with a Hermes leader, tell the leader your
goal, and it builds and coordinates the team. **Leader-driven auto-spawn is a
documented fast-follow** (see "Known limitations") — it depends on a Hermes-side
tool-surfacing fix not yet shipped. Until then, teammates are provisioned with
**one button**, then the leader coordinates them.

### Step 1 — Create the team in AionUi
AionUi → **Teams** → **+** → name it → set the leader to **Hermes Agent** →
create. This gives you a leader-only team.

### Step 2 — Build the team (the obvious button)
Open the **control panel**. At the top is a large **Build Hermes Team** button.
Click it, pick (or type) the team name and the roles you want, and it spawns each
role as a Hermes teammate into that team. The teammate cards appear in the AionUi
panel next to the leader.

> Under the hood this runs `~/beewid-privacy/seed_hermes_team.sh`, which creates
> each teammate via AionUi's own `addAgent` service path (the safe, atomic route —
> not raw DB editing). You can also run it directly:
>
> ```bash
> ~/beewid-privacy/seed_hermes_team.sh "<team name>" "Role 1" "Role 2" ...
> ```
>
> The team must be **open** in AionUi when you run it (the script reads the live
> session's MCP port/token each run).

### Step 3 — Use the team
Click any teammate card to talk to it directly, or prompt the **leader** to
coordinate the team toward a goal. Each teammate is an independent Hermes session.

---

## No-conflict swarm operation

A full fleet of agents coexists without GUI or identity conflicts because of how
teammates are provisioned and isolated:

- **Each teammate gets its own slot + conversation row + isolated ACP session.**
  No shared session state, so agents don't collide in the panel.
- **Stable role identities.** Roles map to stable agent names; the swarm registry
  keeps role→identity assignments fixed so re-provisioning doesn't reshuffle.
- **Workspace boundaries** are honored per the co-work contract (KHAOSS vs Beewid
  ownership, shared import-only modules, vault split). Agents operate in the
  shared team workspace without crossing ownership lines.

This is domain-agnostic: the same Build-Hermes-Team flow works for a trading team,
a research team, a writing team — any lineup of roles you give it.

---

## Patches & durability

The shipping fixes (apply automatically on install; re-run the patcher if wiped):

| Fix | File | Wiped by | Purpose |
|-----|------|----------|---------|
| Team MCP server rename | `TeamMcpServer.ts` (`getStdioConfig` → `name: 'aionui-team'`) | `npm install` | Tools surface as `mcp_aionui_team_team_*` (no per-team UUID), stable across all teams |
| Gemini name-match | `GeminiAgentManager.ts`, `gemini/cli/config.ts` | `npm install` | Auto-approve / allowlist still match the renamed server |
| Lead prompt suffix guidance | `leadPrompt.ts` | `npm install` | Leader knows team tools carry the `mcp_aionui_team_` prefix |
| Hermes MCP capability | `~/.hermes/hermes-agent/acp_adapter/server.py` (`mcp_capabilities=McpCapabilities(stdio=True)` in `initialize()`) | `hermes update` | Hermes advertises stdio MCP so AionUi injects the team server |

> Run the bundled patcher after any `npm install` (AionUi) or `hermes update`.

---

## Known limitations / fast-follow

- **Leader-driven auto-spawn (target: v1.22).** AionUi correctly injects the team
  MCP server into the Hermes leader's session (verified: `loadSession mcpServers=[aionui-team]`),
  but Hermes does not yet *surface* those tools to the model on the resume path —
  so the leader can't call `team_spawn_agent` itself yet. Until fixed, use the
  **Build Hermes Team** button. The fix is a Hermes-adapter change (register the
  injected MCP into the model-visible tool surface on `load_session`/`resume`,
  not just `new_session`). Once it lands, the leader can build its own team from a
  prompt, and a leader skill can auto-propose + spawn a standard lineup.
- **Analyst model credits.** Teammates run on the leader's model provider. If a
  teammate's profile points at a provider with no credit/availability, its tasks
  will fail (e.g. an Anthropic-keyed profile with an empty balance). Point
  teammate profiles at a funded/available provider.

---

## Quirks (for AI-assisted debugging)

1. **Two AionUi installs exist: dev vs packaged.** `~/launch-aionui.sh` runs the
   **dev** build (`electron-vite dev`) — the one that carries the source patches.
   The desktop launcher runs the **packaged** build, which does **not** have the
   patches unless the installer applied them there. Always test with the build
   you actually patched. A "fix didn't work" result is often "tested the wrong
   build."
2. **Blank/white AionUi window — two causes.** (a) `pkill -9 -f electron` can kill
   its own shell because the command line contains "electron"; use a self-safe
   pattern like `pkill -9 -f "[e]lectron"`. (b) Launching electron before the dev
   server finishes compiling main yields a blank renderer; wait for
   "renderer did-finish-load" before interacting. Recovery: kill electron + dev,
   `rm -f ~/.config/AionUi-Dev/Singleton*`, `rm -rf ~/.config/AionUi-Dev/GPUCache/`,
   relaunch via the dev script, wait for the window.
3. **Dead code: `src/process/agent/acp/index.ts` is NOT the live ACP path.**
   `AcpAgentManager` imports it as a type only; the live class is **`AcpAgentV2`**
   (`src/process/acp/compat/AcpAgentV2.ts`), and the session client is
   `ProcessAcpClient`. Patch/instrument those — editing `acp/index.ts` has no
   runtime effect (it's tree-shaken out).
4. **electron-vite dev does not hot-rebuild the main process reliably.** Edits
   under `src/process/**` need a full dev-server restart + electron relaunch to
   take effect. Confirm a change is live with
   `grep -c <marker> out/main/index.js` before trusting a test.
5. **Team tools are namespaced.** After the rename they appear to the model as
   `mcp_aionui_team_team_spawn_agent`, `..._team_members`, etc. — they do **not**
   start with `team_`. To verify an agent has them, ask for its **verbatim** tool
   list (or just have it spawn), not "do you have team_ tools."
6. **Verify by spawning, not by tool-name prefix.** The MCP server registering in
   the backend log does not by itself mean a given chat session can call the tools
   — registration is per-session. Confirm with the actual session's verbatim tool
   list or a live spawn.
7. **Teammate provisioning uses the `addAgent` service path, not raw DB writes.**
   `seed_hermes_team.sh` (via `team_spawn_client.js`) calls the service so the
   teammate's conversation row, slot id, teamId link, and per-slot MCP config are
   created atomically. Raw DB inserts risk a dangling `conversationId` that breaks
   the slot.
8. **Back up `aionui.db` before any DB-level operation:**
   `cp ~/.config/AionUi-Dev/aionui/aionui.db{,.bak.$(date +%s)}`.

---

## Key paths

| Path | Purpose |
|------|---------|
| `~/launch-stack.sh` | Full stack launcher |
| `~/launch-aionui.sh` | AionUi **dev** build launcher (patched build) |
| `~/aionui-stack-panel.html` | Control panel UI (hosts the Build Hermes Team button) |
| `~/beewid-privacy/panel_server.py` | Panel backend (`:7842`) |
| `~/beewid-privacy/seed_hermes_team.sh` | Hermes team provisioner |
| `~/beewid-privacy/team_spawn_client.js` | Node helper the seed script wraps |
| `~/.hermes/hermes-agent/acp_adapter/server.py` | Hermes ACP adapter (mcpCapabilities patch) |
| `~/.config/AionUi-Dev/aionui/aionui.db` | AionUi SQLite (teams, conversations, slots) |
| AionUi source | `/home/boq/Downloads/AIonUi/AionUi-main` |

---

*KHAOSS Stack v1.11 · 🐝 If-Need-Bee*
