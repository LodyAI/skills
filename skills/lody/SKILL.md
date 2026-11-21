---
name: lody
description: Answer questions about using Lody, recommend Lody workflows and best practices, explain workspaces, sessions, tabs, worktrees, Agent Configs, Agent Roles, MCP servers, previews, schedules, automations, team sharing, mobile, and end-to-end encrypted workspaces, and inspect Lody's open-source implementation when exact product details are needed.
---

# Lody User Guide & Product Reference

Use this skill when the user asks how Lody works, how to configure or troubleshoot a Lody feature, or what workflow fits their task in Lody.

## How to answer Lody questions

Resolve answers from the fastest authoritative layer instead of guessing:

1. **Core guide and focused references** — Use this file for Lody's mental model, common distinctions, and recommended workflows. Load only the matching file under `references/` when feature-specific details are needed.
2. **Live runtime and documentation lookup** — Check the running contract whenever exact tool arguments, CLI flags, or recent docs matter:
   - Inside a Lody session: call `lody_help({})` for the MCP tool index and `lody_help({ command, action })` for exact input schemas; use `skill://lody-automations/SKILL.md` when authoring `lody_automation` programs.
   - From a terminal: run `lody <group> [sub] --help` (and use the companion `lody-cli` skill when driving the CLI).
   - Published docs: check `https://lody.ai/llms.txt` (curated index + FAQ) or `https://lody.ai/docs/<slug>` (`https://lody.ai/llms-full.txt` contains the full English docs corpus).
3. **Source-code verification** — When a question hinges on exact UI copy, keyboard bindings, limits, edge-case rules, error codes, or behavior where docs could lag code, follow [references/source-and-troubleshooting.md](references/source-and-troubleshooting.md) to inspect a local Lody checkout or `https://github.com/LodyAI/Lody`. If published docs and current source differ, report what the source implements and note any version caveat.

## Mental model

```text
workspace
├── machines                  private by default; run the Lody CLI daemon (embedded in desktop)
├── projects / repositories   Local Project folder, GitHub repository, or Multi-repo Project Group
│   └── session               one task, one conversation, one branch, and typically one Git worktree
│       ├── round             one agent turn + the file diff it produced
│       ├── child tab         another chat inside the same session sharing its files/worktree
│       ├── side chat         a right-sidebar fork for quick questions alongside the main chat
│       └── pull request      linked GitHub PR with live CI and review-comment sync
├── agent configs             runtime + credentials + environment on a specific machine
│   └── agent roles           reusable presets (target machine + config + model + effort + permission + prompt)
└── mcp servers               shared workspace catalog; credentials stay local to each machine
```

### Editions and workspace modes

- **Standard Cloud Workspace** — Shared across desktop, web, mobile, and CLI once signed in.
- **End-to-End Encrypted (E2EE) Workspace** — Marked with a lock icon. Only authorized devices can read conversations, files, and agent output; new devices require security-code approval and recovery requires a saved recovery file or passkey.
- **Open-Source Local Edition** — Single-machine local workspace catalog (default when building from the public repo) with no cloud account sign-in.

## Key distinctions

| Often confused | How they differ |
| --- | --- |
| **Agent Config vs. Agent Role** | **Agent Config** (`Settings -> Agents`) configures *what* runtime launches on a machine and *how* it authenticates (Claude Code, Codex, Kimi, DeepSeek, ACP, env vars). **Agent Role** (`Settings -> Agent Roles` / `Agent 角色`) saves *how* you run a task: one or more `machineId + agentConfigId` targets, model, reasoning effort, permission mode, optional default instruction, and optional Role Memory (`nmem`). Roles never store API keys or MCP selections. |
| **Child Tab vs. Sub-session** | **Child Tab** (`+` in the session tab bar, or MCP `useCurrentSessionAsParent: true`) lives inside the parent session and shares its machine, directory/worktree, and uncommitted files (one level deep). **Sub-session** (MCP `useCurrentSessionAsParent: false`) is an independent sidebar session nested under its opener (`openedBySessionId`) with its own isolated worktree or project context. |
| **Fork vs. Side Chat** | **Fork session** (below a completed reply when supported by the runtime) copies history up to that reply into either **Current workspace** (shared tab) or **New worktree** (isolated from committed `HEAD`; uncommitted/untracked files stay on the source). **Side Chat** opens a right-sidebar fork on the shared directory for a quick side question without derailing the main thread. |
| **Live Handoff vs. Share Link vs. Export** | **Live Handoff** (`Share with team…` / header access control) lets workspace teammates open and continue the *same* live session, diffs, and worktree; it requires workspace membership plus machine (and local-project, if used) sharing — copying a session URL alone grants no access. **Share Link** (`Settings -> Sharing` / `分享`, or `lody_session_manage` `share`) publishes a static read-only snapshot after consent. **Copy as Markdown / Share Image** exports a static record (requires explicit confirmation in E2EE workspaces). |
| **Lody Auth vs. Provider Auth** | Signing in to Lody authenticates workspace sync and machine registration. Provider credentials belong to the **Agent Config** on the target machine; re-running `lody login` never repairs an expired model API key or ACP login. |

## Recommended workflows and best practices

1. **One atomic task = one Session = one Worktree = one PR**
   - Split features and bug fixes into small, independent sessions rather than overloading one long context window.
   - Use Git-backed **Worktrees** (automatic on GitHub repos and supported on Local Projects) so parallel agents never step on each other's files or branches.
   - Put recurring bootstrap steps (`cp .env.example .env`, `pnpm install`) in **Settings -> Projects -> Worktree setup** so every new worktree starts ready.
2. **Review each Round with Diff Comments**
   - Inspect per-turn or all-changes diffs in the right panel after each round.
   - Add line-level **Diff Comments** (`Cmd/Ctrl+Enter`), send comment threads back into chat for targeted fixes, and enable **Sync to GitHub** on PR-linked sessions so GitHub PR reviews and Lody stay in one loop.
   - For UI work, ask the agent to report the dev server for **Lody Preview**, check responsive viewports, and send element-level **Visual Annotations** directly back to the chat.
3. **Close the PR loop and archive cleanly**
   - Create or link a PR from the session header to monitor live CI checks and review threads; optionally turn on **Auto Delivery** (`autoDeliver`: fix CI, resolve conflicts, merge) and **Archive After Merge** (`archiveAfterMerge`).
   - Archive completed sessions to keep the sidebar focused. Archiving preserves conversation history and commits pending non-ignored changes locally to the session branch before reclaiming a Lody-managed worktree, **but gitignored files (`.env`, build outputs, dependencies) can be discarded** — save needed ignored files outside the worktree first.
4. **Pick the right multi-turn or multi-agent mechanism**

| Goal | Best mechanism |
| --- | --- |
| Run several independent tasks yourself | Open separate sessions in isolated **Worktrees** (or **Child Tabs** when you intentionally want agents sharing uncommitted files). |
| Have one coordinator agent split, steer, and collect work across sessions or machines | **Agent Session Control** (`lody_session_create`, `lody_session_chat`, or `@` mentioning a session / Role). |
| Keep a command running after the current turn ends, or wake the session when a condition becomes true | **Lody Automations** (`lody_automation` `run` / `register`, or session settings `recoverCapacity`, `compactWhenIdle`, `archiveAfterMerge`, `autoDeliver`). |
| Send one message at a future time or right after a Claude 5-hour quota reset | **Schedule send…** (`+` menu in an existing chat) or **Send continue after reset**. |
| Run a prompt on a recurring clock (cron/interval/daily/weekdays) or as a reusable manual trigger | **Scheduled Tasks** (`Schedules` in sidebar, `lody schedule`, or agent-proposed card via `lody_schedule` action `propose`). |

## Reference routing

Load only the reference file that matches the user's question:

| Topic | Reference file |
| --- | --- |
| Composer (`@` mentions, `$` skills, `/` commands, Prompt Shortcuts, drag-and-drop), Message Queue & Steer, Tabs, Worktree scripts, Files/Diffs, Web & Simulator Previews, `lody review`, Embedded Terminal, Zen Mode, Keyboard Shortcuts | [references/sessions-ui-and-collab.md](references/sessions-ui-and-collab.md) |
| Agent Configs & Runtimes (Claude, Codex profiles/forecasts, Kimi, DeepSeek, Devin, Pi, Magpie import, ACP Registry, Custom ACP, Lody Agent), capability refresh, Agent Roles, Role Memory (`nmem`), built-in & custom MCP servers, MCP credential security | [references/agents-roles-and-mcp.md](references/agents-roles-and-mcp.md) |
| Multi-agent Session Orchestration rules & placement, Lody Automations, Scheduled Send & Scheduled Tasks, Team & Machine sharing, GitHub integration, Mobile (iOS Live Activities / Android), E2EE Workspaces & pairing, AI Usage, Quota & Workspace Data export/import | [references/orchestration-schedules-and-team.md](references/orchestration-schedules-and-team.md) |
| Troubleshooting playbook (auth, 3-state machine presence, cache reset vs. hard reset, daemon logs/upgrades, machine identity restore) + **Lody Source Code & Spec Map** for verifying implementation details | [references/source-and-troubleshooting.md](references/source-and-troubleshooting.md) |
