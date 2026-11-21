# Troubleshooting & Lody Source-Code Investigation

## Troubleshooting playbook

### 1. Distinguish Lody account auth from Provider/ACP auth

- **Probe Lody auth safely**: Run a read-only check such as `lody workspace list --json` (`ok: false` + non-zero exit indicates missing/expired Lody credentials).
- **Never run bare `lody login` from an agent**: It waits for browser approval and can hang silently when stdout is not a TTY. Ask the user to run `lody login` in an interactive terminal, or pass a user-supplied CLI API key / machine token via `lody login --auth <key>` over an approved secret channel (never ask users to paste raw secrets into chat).
- **Network outage != bad credentials**: Preserve saved credentials during transient network failures; do not run `logout` or `--hard` cache resets to "fix" connectivity.
- **Provider auth errors**: If Lody commands succeed (`workspace list` works) but a session reports authentication failure, inspect the selected **Agent Config** on its target machine (`Settings -> Agents`, or `lody agent-config show` *without* `--show-secrets` in shared contexts) and complete that provider's sign-in or API key setup.

### 2. Machine status, offline remotes, and single-runtime ownership

- **Three-state machine liveness (`online` | `offline` | `unknown`)**: During cold start or reconnect, a machine whose presence room has not synced yet reports `unknown`, **not** `offline`. Only definite `offline` blocks dispatch; `unknown` proceeds under its own deadline.
- **Desktop app already owns the machine runtime**: Each machine runs one Lody runtime. While the Lody desktop app is open, `lody start` and `lody daemon start` intentionally refuse to start a second runtime — CLI commands automatically share the desktop app's runtime.
- **Teammate cannot see a shared session**: Verify (1) they accepted the workspace invite, (2) the session's machine (and local project, if any) is shared by its owner, and (3) their sidebar filter is set to **All Tasks** (`Team Tasks` on mobile) rather than **My Tasks**.

### 3. Stuck UI, stale capabilities, daemon logs, and machine identity recovery

| Symptom | Resolution |
| --- | --- |
| Newly installed CLI models, modes, or `/` commands missing | Refresh capabilities in **Settings -> Agents** or run `lody agent-config refresh-capabilities <idOrName>` on that online machine. |
| Built-in `lody` MCP fails to connect inside an agent | Check `<Lody data dir>/logs/mcp-http-<date>.log`. Lody sets `NO_PROXY`/`no_proxy` for loopback, so if a failure never reached `mcp-http-*.log`, an agent-internal or OS-level proxy intercepted localhost. |
| Renderer stuck loading or local replica cache corrupted | Use **Clear Cache** (`Settings -> Preferences` or `lody app reset-cache`), which clears rebuildable `lody*` caches on next boot while keeping sign-in, preferences, and E2EE custody. Reserve **Hard Reset** (`lody app reset-cache --hard`) for full wipe + sign-out after user confirmation (check unsynced work and E2EE recovery first). |
| Daemon crash, slow sync, or pending update | Inspect `lody daemon status` and `lody daemon logs -n 50` (logs live under `<Lody data dir>/logs`, default `~/.lody/logs` or `$LODY_DATA_DIR/logs`). Upgrade cleanly via `lody daemon upgrade` + `lody daemon update-status` (waits for idle unless `--immediate`). |
| Reinstalled OS / replaced host needs an old offline machine's identity | Open **Settings -> Machines -> \<owned daemon\> -> Advanced settings -> Restore old machine identity** (note: in E2EE workspaces, changing machine identity still requires fresh approved device pairing). |
| Uncertain whether `session create` or `chat` was accepted after a timeout | Inspect `lody session list`, `status`, and `history` (or `lody_operation` / `lody_session_history` in MCP) before resending — `create` and `chat` do not deduplicate retries without the same Operation ID. |

---

## Investigating Lody's source code when questions get specific

When a user asks about exact implementation details — specific UI copy, validation limits, error codes, keyboard shortcuts, protocol payloads, or behavior not covered above — verify directly in Lody's open-source repository (`LodyAI/Lody`) instead of speculating.

### Step 1: Locate Lody source locally before using the network

Check in order:

1. **Current workspace** — Check whether the current directory already contains `apps/cli`, `packages/components`, and `specs/`.
2. **Local Lody-managed repository cache** — Check `~/.lody/repos/github---lodyai---lody` (or `$LODY_DATA_DIR/repos/github---lodyai---lody`, including its `worktrees/` or `bare.git`) or inspect registered local projects (`lody project list --json`).
3. **Public GitHub repository (`https://github.com/LodyAI/Lody`)** — If no local checkout exists, query `LodyAI/Lody` with `gh` (for example `gh api repos/LodyAI/Lody/git/trees/main?recursive=1` to locate paths and `gh api repos/LodyAI/Lody/contents/<path> -H "Accept: application/vnd.github.raw+json"` to read files) or fetch raw files over HTTPS.

### Step 2: Read in authority order

1. **`specs/*.md` and `specs/e2ee/*.md`** — Product intent, user-visible contracts, edge cases, and protocol rules (many files have `.zh.md` counterparts).
2. **`.agents/docs/*.md` and module `README.md` / `AGENTS.md`** — Cross-module implementation guides, state machines, and invariants.
3. **Executing code + `locales/{en,zh_CN}.json`** — Authoritative schemas, defaults, flags, shortcuts, error strings, and exact English/Chinese UI labels.
4. **`site-docs/content/docs/{en,zh}/`** — Public user documentation MDX source.

### Step 3: Subsystem-to-path map (`LodyAI/Lody`)

| Product area | Paths in `LodyAI/Lody` |
| --- | --- |
| Product contracts & feature specs | `specs/*.md`, `specs/e2ee/*.md` |
| Architecture & state-machine guides | `.agents/docs/*.md` (e.g. `app-state-machines.md`, `sessions-surface.md`, `ui-mentions.md`, `e2ee-workspace-bootstrap.md`) |
| Public docs, changelog & `llms.txt` generator | `site-docs/content/docs/{en,zh}/`, `site-docs/content/changelog/`, `site-docs/scripts/generate-llms.mjs` |
| Exact English & Chinese UI labels | `locales/en.json`, `locales/zh_CN.json` |
| Shared schemas, history, Role/MCP/Schedule contracts | `packages/shared/src/` (`session-data/`, `acp/`, `schedule-control.ts`, `electron-ipc.ts`) |
| Shared React UI (sessions, composer, diffs, sidebar) | `packages/components/src/components/` (`sessions/`, `ai-gui/`, `commands/`, `mobile/`, `harness/`) |
| All Settings pages & tab visibility gates | `packages/components/src/components/settings/` (`settings-tabs.tsx` lists every tab and capability gate) |
| Command palette & keyboard shortcuts | `packages/components/src/lib/commands/` (`shortcuts.ts`, `registry.ts`, `built-ins.ts`) |
| CLI subcommands & flags | `apps/cli/src/commands/` and `apps/cli/src/index.ts` |
| Daemon session lifecycle, Git worktrees & PR poller | `apps/cli/src/session/` (`worktree/`), `apps/cli/src/lib/pr-poller/`, `apps/cli/src/lib/code-collab/` |
| Built-in `lody` MCP server, tool schemas & MCP OAuth | `apps/cli/src/mcp/` (`tool-help.ts`, `authentication/`, `skills/lody-automations/`) |
| Multi-session orchestration & Automations engine | `apps/cli/src/orchestration/`, `apps/cli/src/automations/` |
| ACP runtime adapters, auth & capability probing | `apps/cli/src/agent/`, `packages/acp-extension-{core,claude,codex,devin,dsh,grok,kimi,pi}` |
| Web & Simulator/Emulator previews | `apps/cli/src/preview/`, `apps/cli/src/ios-simulator/`, `apps/cli/src/mcp/simulator-preview-tool.ts` |
| Standalone `lody review` HTML viewer | `packages/code-review-viewer/` |
| Electron desktop shell, menus, IPC & embedded CLI | `apps/electron/src/{main,preload,renderer}/` |
| Native iOS & Android mobile apps | `apps/mobile/` (`ios/`, `android/` submodules) and `apps/mobile-capacitor/` |
| End-to-end encryption & machine-sharing policy | `packages/e2ee-core/`, `packages/e2ee-client/`, `packages/e2ee-machine-sharing/`, `packages/client-e2ee/` |
| Platform capability gates & public cloud DTOs | `packages/platform/`, `packages/cloud-api/` |

### Step 4: Respect the open-source repository boundary

`apps/{cli,electron,mobile}` and the shared `packages/*` above are open source in `LodyAI/Lody`. Hosted cloud backend services, server-side billing/operator configuration, and private web deployment wrappers live outside the public repository (only public protocol names, DTOs, and capability contracts appear in `packages/cloud-api` and `packages/platform`). If a user asks about hosted server internals beyond those public contracts, explain that only the client/daemon/protocol side is in the public repo.
