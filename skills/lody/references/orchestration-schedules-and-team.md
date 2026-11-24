# Orchestration, Automations, Schedules, Team, Mobile & E2EE

## Multi-agent session orchestration (`lody_session_*`)

One session can act as a coordinator that creates, steers, inspects, and collects results from other conversations across agents and machines.

### Placement and worktree behavior (`lody_session_create`)

| Selection | Resulting placement and files |
| --- | --- |
| `useCurrentSessionAsParent: true` | **Child Tab** inside the current session on the same machine; shares the current directory/worktree and uncommitted files. No `workContext` or nested child Tab allowed. |
| `useCurrentSessionAsParent: false`, `workContext.kind: "local"`, `worktree` omitted/`false` | Independent sidebar session running directly in the registered project's original checkout on the target machine. |
| `useCurrentSessionAsParent: false`, `workContext.kind: "local"`, `worktree: true` | Independent sidebar session in a separate Git worktree from the selected committed base (does not copy uncommitted files). |
| `useCurrentSessionAsParent: false`, `workContext.kind: "github"` | Independent sidebar session in its own repository worktree on the target machine. |
| `useCurrentSessionAsParent: false`, `workContext.kind: "chat"` (or omitted without a Role) | Independent plain-chat session with no repository. |

- **Explicit `false` for sidebar sub-sessions**: Same-machine Local Project Role creation defaults to a shared Tab when both `useCurrentSessionAsParent` and `workContext` are omitted. Pass `useCurrentSessionAsParent: false` explicitly whenever you want an independent sidebar sub-conversation.
- **Coordination rules and limits**:
  - Delegated Operations deliver completion callbacks automatically — do **not** poll in a loop, use legacy `wait=true`, or send courtesy/acknowledgement-only replies ("thank you", "got it").
  - During a turn triggered by another session, sending a new chat back to that sender fails with non-retryable `REVERSE_CHAT_FORBIDDEN`; return your answer by finishing your own turn instead.
  - Causal delegation chains cap at **16 hops** (`CHAIN_DEPTH_EXCEEDED`).
  - Messaging a busy session on the calling daemon can steer its active turn (`STEER_DELIVERY_UNKNOWN` means acceptance was uncertain — inspect status/history before resending).
  - A user Stop sets `collaborationStopped`, blocking further delegation until the user resumes.

## Lody Automations (`lody_automation` / `lody automation`)

Automations run on the session's owning machine daemon. For exact condition context APIs and examples, load the built-in MCP skill `skill://lody-automations/SKILL.md`.

- **`run` (outlive the current turn)** — Idle sessions can be reclaimed, terminating ordinary background child processes. `lody_automation` `run` hands a non-shell command to Lody's daemon so it keeps running after the turn ends and queues one continuation on exit (exit code/signal, duration, last 4 KiB of output, and full `logPath` up to 16 MiB). Interrupted runs after a daemon restart are reported, never re-executed automatically.
- **`register` (stateless background condition)** — Evaluates an `async ({ ctx }) => ...` function expression on an interval (`1000`–`86400000` ms, default `30000`). Return `false` to keep waiting or `{ resumeText, payload? }` to resume the session once.
- **`configure` / `settings` (per-session lifecycle automation)**:
  - `recoverCapacity` — Automatically resumes after provider quota/capacity limits clear.
  - `compactWhenIdle` — Compacts context (`contextTokens`, `cacheTtlMs`, `leadTimeMs`) before cache expiry or token pressure while idle.
  - `archiveAfterMerge` — Archives the session once its linked PR merges.
  - `autoDeliver` — Hands PR maintenance (`fixCi`, `resolveConflicts`, optional `merge`) to Lody.

## Scheduled messages and Scheduled tasks

- **One-off Scheduled send** — In an existing conversation, click **`+ -> Schedule send…`** to send text/attachments once at a future time on the target machine's clock (catches up once if offline at the target time).
- **Quota reset continuation** — Session-level **`recoverCapacity`** automation (enabled from the quota-failure banner or `lody_automation` `configure`) automatically resumes the session after provider quota limits clear; where offered above the composer, **Send continue after reset** schedules a one-off `continue` 1 second after a verified Claude 5-hour reset timestamp.
- **Scheduled tasks (`Schedules` in sidebar / `lody://schedules` / `lody schedule` / `lody_schedule`)**:
  - **Execution model**: Owned and executed by **one of your machines** (never in the cloud; does not wake a sleeping computer). Schedule times use **that machine's local time zone**.
  - **Triggers**: `manual` (Run now only), `once`, `interval` / `minutes` / `hours`, `daily`, `weekdays`, `weekly`, `monthly`, or `cron` (CLI).
  - **Destinations**: `new_session` (default), `own_session` (one reusable chat for that schedule), or `existing_session`.
  - **Missed & overlapping runs**: Offline misses run **once** on reconnect (`run_once`, or `skip` in CLI drafts); overlapping runs queue at most one (`queue_one` or `skip`).
  - **Human consent boundary**: Inside an agent session (`LODY_AGENT_SESSION_ID` set), agents can only list/read (`lody_catalog` `schedules`), `pause`, or **propose** (`lody_schedule` action `propose` to create, or `propose_edit` to update title/prompt/rule), posting a confirmation card requiring the user to click **Create** / **Apply** (or share `[New Schedule](lody://schedules?create=1)` when the user prefers filling the editor manually).

## Team collaboration, GitHub, and Mobile

- **Private-by-default resources**: Joining a workspace never exposes your personal machine or local folders. Only the machine owner can toggle **Settings -> Machines -> Share machine** or **Settings -> Projects -> \<project\> -> Team visibility**, or choose **Share with team…** from a session header (`Private`, `Team read-only`, or `Team can edit` when the machine is shared).
- **Public static shares**: Manage, update, reset, or revoke read-only snapshot links in **Settings -> Sharing** (`分享`).
- **GitHub Integration (`Settings -> GitHub`)**:
  - Installing the **Lody GitHub App** grants repository access across workspaces; add authorized repos to each workspace in `Settings -> GitHub`. If an authorized repo is missing, uninstall the Lody GitHub App on GitHub and reinstall it from `Settings -> GitHub -> Manage` with all desired repos selected.
  - Enable **Act as you for PRs, comments, and merges** separately if you want PR actions attributed to your personal GitHub account rather than the app identity.
- **Mobile apps (native iOS & Android)**:
  - Track live output, browse project files without starting a session, review diffs, approve permissions, and read cached chats/diffs offline.
  - **iOS Live Activities**: Shows active agent status on Lock Screen and Dynamic Island, with actionable permission alerts. Enable push in **Settings -> Preferences -> Push Notifications**.

## End-to-End Encrypted (E2EE) workspaces

- **Security model**: Content keys stay on authorized devices; a signed permission ledger prevents silent member/device injection. Verify membership integrity under **Settings -> General -> Open encryption console -> Security code & history**.
- **Device & member onboarding**: Opening an E2EE workspace on a new device or browser requests approval from an already authorized device (match the displayed numbers before clicking **Authorize**).
- **Mandatory recovery setup**: Configure a **Recovery file** or **Passkey** under **Open encryption console -> Recovery**. If every authorized device is lost without recovery, Lody cannot decrypt the workspace.
- **Headless machine pairing without account login**: Claim an expiring desktop offer on the server with `lody machine pair --offer <cmd>`, then run `lody start --paired-workspace <account/workspace>`. Never echo pairing offers in chat or reuse a claimed offer.

## Usage, quota, and workspace data

- **AI Usage (`Settings -> AI Usage` / `AI 用量`)**: Token and estimated-cost charts (24h, 7d, 30d, all time; hourly bars use device timezone while calendar/day detail uses UTC), broken down by model and member, plus shareable usage summary cards.
- **Quota & device resources**: Check subscription rate-limit windows (e.g., 5h / 7d) and reset forecasts under **Settings -> Agents**, and live machine CPU/memory/disk in **Settings -> Machines** (or `lody_catalog` `machines` with `includeResources: true`).
- **Workspace export & import**: Owners/admins can export or merge complete workspace bundles in **Settings -> Data** (desktop) or via `lody workspace export --out <file>` / `lody workspace import <file>` (**warning**: export bundles are plaintext and include every conversation and attachment). Use `lody export [dir]` to write readable session files locally.
