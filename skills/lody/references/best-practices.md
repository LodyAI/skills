# High-Leverage Lody Best Practices

Share these recommendations when the user asks for workflow advice, power-user habits, or how to get the most out of Lody.

## 1. Choose the container by its context-isolation boundary

Avoid running unrelated questions, implementation, and exploratory rewrites inside one long conversation. Match the Lody container to what should and should not be shared:

- **Side question or risk check -> `Side Chat` (then close it)** — When you want to ask "why is this structured this way?" or "could changing X break Y?" mid-task, open a **Side Chat** in the right sidebar instead of asking in the main chat. Long explanatory tangents stay out of the main context window.
- **Second opinion on uncommitted edits -> `Child Tab` with a different model** — Click `+` in the session tab bar to open a Child Tab sharing the exact same worktree and uncommitted files, then switch that tab to a different provider/model (for example, Claude implementing in the main tab while high-reasoning Codex reviews or writes edge-case tests in the child tab). Both tabs see the same disk state without sharing confirmation bias or polluting each other's transcript.
- **Competing approaches -> `Fork session` into a `New worktree`** — At a design fork, branch from the current reply into a separate worktree and compare actual diffs rather than repeatedly rolling back one checkout.
- **Independent feature or bug fix -> separate `Session` (`Worktree`)** — Keep one atomic outcome per session and PR so human review and context stay small.

## 2. Build a small, complementary `Agent Role` team

Instead of writing generic persona prompts, save 3–4 Roles (`Settings -> Agent Roles`, `[New Agent Role](lody://settings/agent-roles?create=1)`) that combine distinct model strengths, reasoning tiers, and permission guardrails:

- **Read-only explorer (`@scout`)** — Read-only permissions + a model strong at long-context codebase tracing, used to map call chains and root causes before editing.
- **Primary implementer (`@builder`)** — Edit/command permissions enabled on your preferred coding runtime.
- **Adversarial reviewer (`@critic`)** — Bound to a *different* model family at high reasoning effort, with a concise default instruction focusing strictly on ship-blocking bugs, data loss, and security/permission boundaries.

Invoke Roles from the composer picker or inline with `@role-name` (or let the agent propose one directly in chat via `lody_agent_role` `create`).

## 3. Use "1 Coordinator Session + N Delegated Sub-sessions" for large work

Opening 8 parallel sessions manually creates heavy human context-switching overhead. For cross-module refactors, batch migrations, or multi-repo changes:

1. Stay in **one main Coordinator session** to align on interfaces, constraints, and acceptance criteria.
2. Ask the Coordinator agent to spawn and steer focused sub-sessions in parallel via Lody's built-in MCP tools (`lody_session_create` with `useCurrentSessionAsParent: false`, `lody_session_chat`), potentially assigning different Roles or machines to each.
3. Let delegated completion callbacks return automatically to the Coordinator, so you review one synthesized summary and the final diffs rather than micromanaging every worker thread.

## 4. Feed spatial anchors instead of prose descriptions

Eliminate vague natural-language location descriptions ("move the button in the top right", "fix the check in the second function"):

- **Code feedback** — Review round diffs in the right panel, attach multiple line-level **Diff Comments** (`+` beside line numbers), and send them back together in one batch rather than interrupting the agent five separate times.
- **UI feedback** — Open **Lody Preview**, switch on annotation mode, and click the exact DOM element to send **Visual Annotations** straight into chat.
- **Prior conversation context** — Drag a related session from the sidebar into the composer (`@Session`) so the current agent reads exact evidence via `lody_session_history` instead of relying on copy-pasted excerpts.

## 5. Zero out parallel-startup friction with `Worktree setup` scripts

Parallel worktrees only feel effortless when a fresh checkout can build and run tests on turn 1 without spending three turns fixing missing `.env` files or dependencies.

- When onboarding a primary repository or local folder, configure **Settings -> Projects -> \<project\> -> Worktree** (`[Open Project Settings](lody://settings/projects)`) with an idempotent **Worktree setup** script (for example copying/linking local env templates and running fast cached package installs).

## 6. Offload tail-end waiting to `Automations` and use Mobile as an approval valve

The slowest 20% of engineering work is waiting on long test suites, CI checks, merge conflicts, or quota resets:

- **Automate the tail** — Use `lody_automation` `run` for commands that must outlive the current turn, enable **`autoDeliver`** (`fixCi`, `resolveConflicts`, optional `merge`) and **`archiveAfterMerge`** once a PR is open, and turn on **`recoverCapacity`** so quota windows resume unattended.
- **Keep compute on an always-on host, use Mobile to unblock** — Run `lody daemon` on a workstation or remote dev box (`[Machines](lody://settings/machines)`). Use desktop for high-bandwidth architecture and diff review; when away from your desk, use the native mobile app (push notifications + iOS Lock Screen / Dynamic Island **Live Activities**) strictly to approve permission prompts and unblock directional questions so background agents never stall waiting on you.

## 7. Replace screenshot debugging with `Live Handoff`

When stuck on a subtle bug or handing work across time zones/roles (e.g., frontend to backend), avoid pasting screenshots or truncated logs into external chat apps. Use **Share with team…** in the session header so your teammate opens the **same live session, diffs, and worktree** — they can inspect the full reasoning trail, ask a quick question in a Side Chat, or send the next prompt directly in place.
