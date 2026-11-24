# Sessions, Composer, Previews, Terminal & Shortcuts

## Composer triggers and rich inputs

- **`@` Mentions** — Type `@` in the composer to reference:
  - **Files** — Fuzzy-matches filenames and paths and injects full-path context. On the Electron desktop app, dragging local files or folders from the OS inserts path mentions for folders (and files where applicable); web and mobile cannot resolve dropped OS folder paths.
  - **Sessions** — Inserts a conversation reference (`[@Title](lody://session/<id>?workspace=<id>)`) so the current agent can inspect or message another session (including across machines when authorized). On desktop, you can also drag a sidebar session row or tab directly into the composer. Include an explicit instruction telling the agent what to read or coordinate.
  - **Issues & Pull Requests** — Available after selecting a GitHub repository; inserts `#<number>` and injects the Issue/PR context.
  - **Agent Roles** — Mentioning `@role-name` instructs the agent to delegate a *new* conversation using that Role's frozen target, run configuration, and default instruction. Local projects and plain chats offer Roles on the same machine; GitHub projects also offer Roles on your other authorized machines.
  - **Skills** — Also reachable directly via `$`.
- **`$` Skill Mentions** — Type `$` to filter discovered **Project Skills** (from the local project or a GitHub repo's default branch) and **Global Skills** (from compatible directories on the target machine, such as `~/.codex/skills` or `~/.claude/skills`). Skills are read-only in Lody: selecting `$skill-name` expands the corresponding `SKILL.md` path when sent. Browse discovered skills under **Settings -> Projects -> \<project\> -> Skills** (mobile: **Project -> Settings -> Project skills**).
- **`/` Slash Commands & Prompt Shortcuts** — Type `/` as the first character of a turn to search two sources together:
  - **Agent commands** advertised by the selected Agent Config (such as `/review`, `/compact`, `/undo`). Note: manually running `/compact` in Claude Code or Codex currently does not render a corresponding UI reply update in Lody; for unattended context management, consider session `compactWhenIdle` automation where supported.
  - **Workspace Prompt Shortcuts** configured in **Settings -> Prompt Shortcuts**.
- **Attachments and agent-published files** — Attach images or files from UI or CLI (`--attach <path>`, up to 8 non-empty files, at most 100 MiB each; atomic — if one fails, nothing is sent). Inside a session, agents publish inline images or downloadable files only when requested via MCP `lody_upload` (`kind: "images" | "files"`).

## Turn control, queueing, quotes, and tabs

- **Message Queue & Steer** — Sending while a turn is running places messages in the session queue above the composer, where you can inspect, edit/replace, withdraw, or steer the active turn (`lody_session_queue` / `lody_session_chat`). Pressing Stop retains paused queue drafts rather than discarding them.
- **Quotes & Ask-Question notes** — Quote specific spans of earlier messages into a reply (requires target daemon `messageQuotes` support), and attach free-text notes when answering structured agent multiple-choice questions.
- **Session Tabs** — Click `+` in the session tab bar to open child chats sharing the parent's machine, branch, worktree, and uncommitted files, or open file and diff tabs beside chat:
  - Drag tabs to reorder (persisted per browser); double-click a chat tab to rename.
  - Closing a conversation tab sets `isTabClosed` across the workspace without stopping the running agent, archiving the session, or deleting its worktree.
  - Reopen closed (and archived) tabs from the tab bar's history button.
- **Sidebar views & bulk actions** — Switch between project-grouped view (**Hide Empty Projects** option; drag handles reorder repo groups) and **Updated** recency view (Today, This week, Older). Filter between **My Tasks** and **All Tasks** (**Team Tasks** on mobile). Hover a session checkbox in Archive to bulk-select for restore or permanent deletion.

## Worktrees and lifecycle scripts

- **Layout** — Lody stores managed Git worktrees under `<Lody data dir>/repos/<repoId>/{bare.git,cache,worktrees/<sessionId>}` (default branch `session/<shortSessionId>`, which can be renamed).
- **Archive & delete retention**:
  - **Archiving** keeps conversation history and the local Git branch, stages and commits pending **non-ignored** changes locally, runs any configured cleanup script *before* that backup commit, and lets the owning machine reclaim the managed worktree directory. Original Local Project directories are never deleted by managed-worktree cleanup.
  - **Restoring** a cleaned-up session recreates its worktree from the preserved local branch and re-runs the setup script; gitignored files (`.env`, build artifacts) must be recreated by the setup script or manually.
  - **Permanent deletion** (from the archive list) removes conversation history irreversibly while keeping the local Git branch and commits — except for projectless plain-chat sessions, where deletion also removes the managed chat working directory and its files.
- **Worktree setup & cleanup scripts** — Configure in **Settings -> Projects -> \<project\> -> Worktree setup / Worktree cleanup** (Bash on macOS/Linux/WSL; PowerShell on Windows; GitHub repos accept both).
  - Exported variables inside a script live only for that script; put agent-runtime env vars in **Settings -> Agents**.
  - Injected env vars (working directory defaults to `LODY_WORKTREE_PATH`): `LODY_WORKSPACE_ID`, `LODY_SESSION_ID`, `LODY_WORKTREE_PATH`, `LODY_WORKTREE_BRANCH`, `LODY_WORKTREE_SCRIPT_PHASE` (`setup` or `cleanup`), plus `LODY_GITHUB_REPO` (GitHub projects) or `LODY_LOCAL_PROJECT_ID` (local projects).

## Files, diffs, previews, review reports, and terminal

- **Files & Editor** — Browse the session file tree in the right panel. Small workspace text files open in the editable viewer (`Mod+S` to save); large files open in a bounded paged read-only viewer; paths outside the workspace open read-only. On desktop local sessions, use **Open in** in the header to launch VS Code, Zed, Xcode, Sublime Text, or custom templates (`Settings -> Preferences -> Path launchers` using `{path}`).
- **Diff Viewer & Diff Comments** — Inspect single-round or full-session (`Changes`) diffs. Hover a line number and click `+` (or tap on mobile) to add Markdown **Diff Comments** (`Cmd/Ctrl+Enter`). Reply, **Resolve**/**Unresolve**, send a thread back to chat, or turn on **Sync to GitHub** on PR-linked sessions.
- **Lody Preview (Web)** — When an agent starts a dev server and reports it via `lody_report_preview_candidate`, open the **Preview** / **Browser** panel:
  - Local desktop sessions can use **Design mode** directly without a tunnel.
  - Remote or shared sessions use **Preview Remote -> Create connection**.
  - Includes desktop/tablet/mobile viewports, rotation, zoom/pan, share URL copy, and **Visual Annotations** (click any UI element to attach a comment and send the element target + feedback into chat).
- **Simulator & Emulator Preview** — Preview running iOS Simulators (macOS hosts) or Android Emulators inside Lody via `lody_simulator_preview` (`list`, `start`, `status`, `stop`). The tool streams/controls device preview state; building and installing the app remains separate.
- **Lody Review (`lody review`)** — Renders a `.review.md` or `.review.json[.gz|.br]` code review into a standalone local `file://` HTML report without requiring Lody sign-in:
  1. `npx lody review prompt` (prints the `.review.md` authoring specification)
  2. Have the agent write `<name>.review.md` against locally fetched commits
  3. `npx lody review <name>.review.md [--repo <dir>] [--output <out.html>] [--no-open]`
- **Embedded Terminal** — Available on the Electron desktop app for non-archived sessions owned by the local machine (`Ctrl+\`` or `Mod+J`, up to 8 tabs). `Alt+N` while focused opens another terminal tab. Processes persist across UI reloads while the local daemon runs and close on archive/delete or daemon stop. Terminal font is configurable under **Settings -> Appearance**.

## Command palette, Zen mode, and default shortcuts

Customize per-device bindings in **Settings -> Keyboard** (`快捷键`). `Mod` is `Cmd` on macOS and `Ctrl` on Windows/Linux (`packages/components/src/lib/commands/shortcuts.ts`):

| Action | Default binding | Scope / notes |
| --- | --- | --- |
| Command Palette & session switcher | `Mod+K` or `Mod+Shift+P` | Searches available commands and session titles/repos/branches (not message bodies) |
| Search inside current conversation | `Mod+F` (desktop) / `Mod+Alt+F` (web) | Searches conversation text |
| Open Settings | `Mod+,` | Desktop and web |
| Toggle Zen Mode | `Mod+.` | Hides/restores sidebars; yields inside Monaco editor for Quick Fix |
| Toggle left / right explorer sidebar | `Mod+B` / `Mod+Alt+B` | Opening any right panel exits Zen Mode |
| New session | `Mod+N` (desktop) / `Mod+Alt+N` (web) | Opens home composer |
| New session tab (or terminal tab when terminal focused) | `Alt+N`, plus `Mod+T` on desktop | Desktop/web (`Mod+T` desktop only) |
| Previous / next tab in session | `Mod+Shift+,` / `Mod+Shift+.` | Desktop |
| Previous / next visible sidebar session | `Mod+Shift+[` / `Mod+Shift+]` | Desktop |
| Jump to visible sidebar sessions 1–9 | `Mod+1` .. `Mod+9` | Desktop; selects across expanded sidebar sections |
| Close active tab or window | `Mod+W` | Native desktop menu accelerator (not rebindable in settings) |
| Focus composer / cycle agent mode | `Mod+L` (desktop) / `Shift+Tab` (in composer) | Provider/model/effort cyclers are unbound by default but rebindable |
| Toggle embedded terminal | `Ctrl+\`` or `Mod+J` | Desktop local sessions |
| Archive / Pin / Rename current session | `Mod+Alt+A` / `Mod+Alt+P` / `F2` | Desktop and web |
| Copy current branch / session URL | `Alt+Shift+B` / `Alt+Shift+C` | Desktop and web |
| Bring Lody to Front (OS global shortcut) | `Mod+Shift+L` | System-wide on desktop (`Settings -> Keyboard -> Global shortcuts`) |
