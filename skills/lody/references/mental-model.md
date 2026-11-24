# Core Mental Model & Editions

## Object hierarchy

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

A **Session** is the primary unit of work in Lody: projects, Agent Configs, Agent Roles, and MCP servers feed into a session, while rounds, diffs, previews, child tabs, side chats, and pull requests happen inside or come out of it.

## Editions and workspace modes

- **Standard Cloud Workspace** — Shared across desktop, web, mobile, and CLI once signed in.
- **End-to-End Encrypted (E2EE) Workspace** — Marked with a lock icon next to the workspace name. Only authorized devices can read conversations, files, and agent output; new devices require security-code approval and recovery requires a saved recovery file or passkey.
- **Open-Source Local Edition** — Single-machine local workspace catalog (the default when building from the public repository) with no cloud account sign-in.
