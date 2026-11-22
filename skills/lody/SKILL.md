---
name: lody
description: Answer questions about using Lody, explain workspaces, sessions, tabs, worktrees, Agent Configs, Agent Roles, MCP servers, previews, schedules, automations, team sharing, mobile, and end-to-end encrypted workspaces, surface actionable in-app links and proposal cards, and inspect Lody's open-source implementation when exact product details are needed.
---

# Lody User Guide & Product Reference

Use this skill when the user asks how Lody works, how to configure or troubleshoot a Lody feature, or where a capability lives in Lody.

## How to answer Lody questions

Resolve answers from the fastest authoritative layer instead of guessing:

1. **Core guide and focused references** — Use this file for common distinctions, actionable reply conventions, and topic routing. Load only the matching file under `references/` when feature-specific details are needed.
2. **Live runtime and documentation lookup** — Check the running contract whenever exact tool arguments, CLI flags, or recent docs matter:
   - Inside a Lody session: call `lody_help({})` for the MCP tool index and `lody_help({ command, action })` for exact input schemas; use `skill://lody-automations/SKILL.md` when authoring `lody_automation` programs.
   - From a terminal: run `lody <group> [sub] --help` (and use the companion `lody-cli` skill when driving the CLI).
   - Published docs: check `https://lody.ai/llms.txt` (curated index + FAQ) or `https://lody.ai/docs/<slug>` (`https://lody.ai/llms-full.txt` contains the full English docs corpus).
3. **Source-code verification** — When a question hinges on exact UI copy, keyboard bindings, limits, edge-case rules, error codes, or behavior where docs could lag code, follow [references/source-and-troubleshooting.md](references/source-and-troubleshooting.md) to inspect a local Lody checkout or `https://github.com/LodyAI/Lody`. If published docs and current source differ, report what the source implements and note any version caveat.

## Key distinctions

| Often confused | How they differ |
| --- | --- |
| **Agent Config vs. Agent Role** | **Agent Config** (`Settings -> Agents`) configures *what* runtime launches on a machine and *how* it authenticates (Claude Code, Codex, Kimi, DeepSeek, ACP, env vars). **Agent Role** (`Settings -> Agent Roles` / `Agent 角色`) saves *how* you run a task: one or more `machineId + agentConfigId` targets, model, reasoning effort, permission mode, optional default instruction, and optional Role Memory (`nmem`). Roles never store API keys or MCP selections. |
| **Child Tab vs. Sub-session** | **Child Tab** (`+` in the session tab bar, or MCP `useCurrentSessionAsParent: true`) lives inside the parent session and shares its machine, directory/worktree, and uncommitted files (one level deep). **Sub-session** (MCP `useCurrentSessionAsParent: false`) is an independent sidebar session nested under its opener (`openedBySessionId`) with its own isolated worktree or project context. |
| **Fork vs. Side Chat** | **Fork session** (below a completed reply when supported by the runtime) copies history up to that reply into either **Current workspace** (shared tab) or **New worktree** (isolated from committed `HEAD`; uncommitted/untracked files stay on the source). **Side Chat** opens a right-sidebar fork on the shared directory for a quick side question without derailing the main thread. |
| **Live Handoff vs. Share Link vs. Export** | **Live Handoff** (`Share with team…` / header access control) lets workspace teammates open and continue the *same* live session, diffs, and worktree; it requires workspace membership plus machine (and local-project, if used) sharing — copying a session URL alone grants no access. **Share Link** (`Settings -> Sharing` / `分享`, or `lody_session_manage` `share`) publishes a static read-only snapshot after consent. **Copy as Markdown / Share Image** exports a static record (requires explicit confirmation in E2EE workspaces). |
| **Lody Auth vs. Provider Auth** | Signing in to Lody authenticates workspace sync and machine registration. Provider credentials belong to the **Agent Config** on the target machine; re-running `lody login` never repairs an expired model API key or ACP login. |

## Actionable replies: Proposal Cards and `lody://` Action Links

Whenever a user asks how to configure, open, or set up something in Lody, make the answer directly actionable instead of prose-only:

1. **Stage in-chat confirmation cards first when Lody MCP supports it** (inside a Lody session, after reading `lody_help`):
   - **Agent Roles** — Call `lody_catalog` (`machines`, `agent_configs`) to resolve IDs, then call `lody_agent_role` (`create` or `update`) so the user can click **Save** directly on the review card.
   - **Scheduled Tasks** — Call `lody_schedule` (`action: "propose"`) to post a **Schedule this task?** card in chat.
   - **Workspace MCP Servers** — On an explicit user request, call `lody_mcp_configure` to create the unselected catalog entry, then include `[Authenticate & Enable in MCP Settings](lody://settings/mcp?server=<id>)` so one click opens that server's local keyring/OAuth editor.
2. **Pair UI breadcrumb paths with clickable `lody://` Action Links** — Always write both the human-readable menu path and the Markdown link so older clients and external terminals stay readable while Lody renders an interactive chip that opens the exact modal/dialog in place:
   - Example: `Open **Settings -> Agents** ([Configure Codex](lody://settings/agents?create=codex))`
   - When inside a Lody session and targeting a machine-scoped surface (`agents`, `agent-roles`, `machines`, `memory`, `projects`), inspect `lody_catalog` first when practical so you can append `&machine=<machineId>` (or `?config=<agentConfigId>`).

| Destination | Canonical `lody://` URI | Notes & allowed parameters |
| --- | --- | --- |
| Create Agent Config (pre-selected) | `lody://settings/agents?create=<target>[&machine=<id>]` | `<target>`: `codex`, `claude`, `kimi`, `deepseek`, `pi`, `devin`, `grok`, `antigravity`, `bub`, `dimcode`, `glm-over-claude-code`, `mimo-over-claude-code`, `minimax-over-claude-code`, `custom`, or `registry:<id>` (e.g. `registry:opencode`) |
| Edit / refresh existing Agent Config | `lody://settings/agents?config=<configId>[&machine=<id>]` | Opens that config's edit & capability-refresh dialog |
| Agent Roles | `lody://settings/agent-roles[?create=1 \| ?role=<id>][&machine=<id>]` | Opens Role list, new-Role dialog, or an editable Role |
| MCP Servers | `lody://settings/mcp[?create=1 \| ?server=<id>]` | Opens MCP list, add dialog, or a specific server's editor/auth |
| Projects, Worktree scripts & Skills | `lody://settings/projects` | Add folder/group: `?add=local[&machine=<id>]` or `?add=group`. Open project tab: `?repo=<owner/name>[&page=<p>]` or `?machine=<id>&local=<projectId>[&page=<p>]` (`<p>`: `general`, `worktree`, `skills`, `defaultMcp`, or local-only `conversations`) |
| Prompt Shortcuts | `lody://settings/prompt-shortcuts[?create=1]` | Opens Prompt Shortcuts or its new-shortcut editor |
| Scheduled Tasks | `lody://schedules[?create=1 \| ?schedule=<id>]` | Opens Schedules list, new-schedule editor, or a schedule's detail |
| Specific Session | `lody://session/<sessionId>[?workspace=<workspaceId>]` | Renders as a conversation chip (`[@Title](lody://session/...)`) |
| Other Settings tabs | `lody://settings/<tab>[?workspace=<id>]` | `<tab>`: `machines` (`&machine=<id>`), `memory` (`&machine=<id>`), `github`, `preferences`, `appearance`, `keyboard-shortcuts`, `workspace` (members & encryption console button), `shares`, `ai-usage`, `billing`, `data`, `account`, `about` |

- **Strict validation boundary**: Every `lody://` link above also accepts optional `workspace=<workspaceId>`. Never add unlisted parameters (`command`, `url`, `env`, `apiKey`, `token`, `prompt`, trailing slashes, or `#fragments`) — Lody's parser rejects non-allowlisted query keys and strips invalid links for security.

## Reference routing

Load only the reference file that matches the user's question:

| Topic | Reference file |
| --- | --- |
| Core mental model (workspace, machine, project, session, round hierarchy) and workspace editions (Cloud, E2EE, Open-Source Local) | [references/mental-model.md](references/mental-model.md) |
| Composer (`@` mentions, `$` skills, `/` commands, Prompt Shortcuts, drag-and-drop), Message Queue & Steer, Tabs, Worktree scripts, Files/Diffs, Web & Simulator Previews, `lody review`, Embedded Terminal, Zen Mode, Keyboard Shortcuts | [references/sessions-ui-and-collab.md](references/sessions-ui-and-collab.md) |
| Agent Configs & Runtimes (Claude, Codex profiles/forecasts, Kimi, DeepSeek, Devin, Pi, Magpie import, ACP Registry, Custom ACP), capability refresh, Agent Roles, Role Memory (`nmem`), built-in & custom MCP servers, MCP credential security | [references/agents-roles-and-mcp.md](references/agents-roles-and-mcp.md) |
| Multi-agent Session Orchestration rules & placement, Lody Automations, Scheduled Send & Scheduled Tasks, Team & Machine sharing, GitHub integration, Mobile (iOS Live Activities / Android), E2EE Workspaces & pairing, AI Usage, Quota & Workspace Data export/import | [references/orchestration-schedules-and-team.md](references/orchestration-schedules-and-team.md) |
| Troubleshooting playbook (auth, 3-state machine presence, cache reset vs. hard reset, daemon logs/upgrades, machine identity restore) + **Lody Source Code & Spec Map** for verifying implementation details | [references/source-and-troubleshooting.md](references/source-and-troubleshooting.md) |
