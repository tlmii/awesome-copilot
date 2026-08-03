---
title: 'Managing Multiple Sessions in the Copilot CLI'
description: 'Learn how to run and manage multiple concurrent Copilot CLI sessions, use the Sessions sidebar, switch between worktrees, and work on parallel tasks efficiently.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-03
estimatedReadingTime: '6 minutes'
tags:
  - copilot-cli
  - sessions
  - multi-session
  - worktrees
relatedArticles:
  - ./using-copilot-coding-agent.md
  - ./github-copilot-app.md
  - ./copilot-configuration-basics.md
prerequisites:
  - GitHub Copilot CLI installed (v1.0.76+)
  - Basic familiarity with GitHub Copilot CLI sessions
---

The Copilot CLI lets you run multiple sessions in parallel from a single terminal window. Each session maintains its own conversation history, working directory, and agent context — so you can have one session debugging a bug while another researches an API design, without losing your place in either.

This guide explains how to manage multiple sessions with the Sessions sidebar, how to create new sessions in isolated worktrees, and best practices for parallel work.

## The Sessions Sidebar

The Sessions sidebar is an experimental panel that shows all your active sessions at a glance. You can switch between them, spawn new ones, and monitor their status without leaving the CLI.

### Enabling the Sessions Sidebar

The sidebar is opt-in while it's experimental:

```
/experimental on
```

Once enabled, the sidebar appears as a split-view panel. You'll see each active session listed with its current status.

### Navigating the Sidebar

| Action | How |
|--------|-----|
| Switch to a session | Click or press the session in the sidebar |
| Open a new session | Select "New Session" from the sidebar |
| See session status | Status indicator next to each session name |

The active session is highlighted (accented by default — opt out with `sidebar.accentActiveSession: false` in your settings). Hover-to-focus is off by default; turn it on with `sidebar.hoverFocus: true`.

### Session Independence

Each session in the sidebar is fully independent:

- Switching sessions **does not restart MCP servers** or rebuild hook state
- A turn running in one session is never interrupted when you switch to another
- Unsent prompt text stays with the session it was typed for

## Creating Sessions in New Worktrees

The `/new-worktree` command creates a git worktree for a new branch and starts a fresh session in it. This is ideal when you need to work on two tasks simultaneously without switching branches in your main checkout.

```
/new-worktree
```

This opens a new session rooted in the new worktree. The original session keeps working in its own branch unaffected.

### Why Worktrees Matter for Parallel Work

A git worktree is a real, separate working copy of your repository — like having the repo checked out twice on disk simultaneously. Combined with sessions, this means:

- No stash/unstash cycles when context-switching
- Each session's file edits stay isolated to its own branch
- You can run builds and tests in parallel in each worktree

This is the same isolation model the [GitHub Copilot app](../github-copilot-app/) uses for its parallel agent sessions, but available directly from the CLI.

## Switching and Resuming Sessions

### Switching Between Active Sessions

From inside any session, you can switch to another using the sidebar (when experimental mode is on) or by opening a new window via the session picker.

When you resume a session, **its autopilot or plan mode is restored** automatically. So if a session was running in autopilot when you left, it will still be in autopilot when you return — the `task_complete` tool stays available and the mode matches what you left.

### Resuming a Previous Session

To resume a session from a previous CLI run:

```bash
copilot --resume
```

The `--resume` picker shows all sessions you can reconnect to, including cloud agent sessions that haven't yet pushed any changes to their branch.

## Controlling Approval Modes

Each session has an **approval mode** that controls how much the agent can do autonomously. Switch modes at any time with:

```
/permissions
```

This opens an interactive dialog to choose between:

- **Interactive**: You approve each tool use before it runs
- **Plan**: The agent creates a plan and shows it before doing anything
- **Autopilot**: The agent proceeds without asking for approval (subject to sandbox policy)

You can also change the mode when launching a session:

```bash
copilot --mode autopilot
copilot --mode plan
```

> **Tip**: In autopilot mode, `task_complete` is available to the agent. If you switch away from a session in autopilot and come back later, the mode is restored so you can pick up right where the agent left off.

## Managing Queued Messages

If you want to send several follow-up messages to an agent while it's working, you can queue them up. The **queue manager** (available from the prompt area) lets you:

- **Reorder** messages before they're sent
- **Edit** a queued message
- **Remove** a message you no longer want sent
- **Repeat** a previously sent message
- **Send immediately** — skip the queue and send now

This is especially useful in autopilot sessions where you're steering a long-running task: queue your next instructions while the agent is still working, then let it pick them up automatically.

## Tracking Credit Usage Across Sessions

When running many parallel sessions, it can be useful to estimate how much AI credit a session will consume before starting. Use `/limits predict` to get a suggestion based on similar past sessions:

```
/limits predict
```

The CLI analyzes your recent session history to estimate credit usage for the current task, helping you make informed decisions about which work to parallelize.

## Best Practices

### When to Use Multiple Sessions

| Scenario | Approach |
|----------|----------|
| Fix a bug while writing a feature | Two sessions in separate worktrees via `/new-worktree` |
| Research an API while implementing it | Two sessions on the same branch — one for research, one for code |
| Run a long autopilot task in the background | Autopilot session + interactive session in the sidebar |
| Monitor a remote coding agent session | Sidebar + `/remote on` to watch the cloud agent |

### Keeping Sessions Organized

- Give sessions a clear initial prompt so you can identify them in the sidebar
- Use `/new-worktree` for sessions that involve file changes — this prevents branches from interfering
- Use `/limits predict` before starting a large parallel batch to estimate costs

### Session Performance

If you're resuming sessions with long histories, the CLI reads the transcript once at startup in parallel — large sessions resume in well under a second even for very long histories.

## Further Reading

- [Using the Copilot Coding Agent](../using-copilot-coding-agent/) — remote control and cloud agent sessions
- [Getting Started with the GitHub Copilot app](../github-copilot-app/) — parallel sessions with the desktop app
- [Copilot CLI releases](https://github.com/github/copilot-cli/releases) — Sessions sidebar added in v1.0.76; `/new-worktree` added in v1.0.78

---
