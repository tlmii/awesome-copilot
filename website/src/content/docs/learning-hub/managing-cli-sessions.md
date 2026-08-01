---
title: 'Managing CLI Sessions'
description: 'Learn how to manage multiple concurrent Copilot CLI sessions, control approval modes with /permissions, predict session limits, and log in with the new browser-based OAuth flow.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-01
estimatedReadingTime: '7 minutes'
tags:
  - copilot-cli
  - sessions
  - configuration
  - fundamentals
relatedArticles:
  - ./copilot-configuration-basics.md
  - ./agents-and-subagents.md
  - ./installing-and-using-plugins.md
prerequisites:
  - GitHub Copilot CLI installed and authenticated
---

As your Copilot CLI usage grows, you'll often want to run multiple tasks in parallel, control how much autonomy the agent has, and stay within your AI credit budget. This article covers three important CLI capabilities for power users: the **Sessions sidebar**, the **`/permissions` command**, and the **`/limits predict`** tool.

## Multiple Concurrent Sessions

Copilot CLI supports running multiple sessions at once — for example, fixing a bug in one session while generating tests in another. The Sessions sidebar, available in **experimental mode**, gives you a single interface to manage all of them.

### Enabling the Sessions Sidebar

Enable experimental mode first, then the sidebar becomes available:

```
/experimental on
```

The sidebar opens on the side of your terminal. From here you can:

- **Switch between sessions** without closing and reopening the CLI
- **Spawn new sessions** for parallel tasks
- **See each session's status** at a glance (thinking, waiting, complete)

### Working Across Sessions

When the sidebar is open, you can have one session run a long-running agent task while you interact with another. Switching sessions does **not** restart MCP servers or rebuild hook state, so tasks in other sessions continue uninterrupted.

> **Tip**: Unsent prompt text stays with the session you typed it in, not the session you switch to. Your draft is always where you left it.

### Session Persistence

Sessions persist across CLI restarts. When you resume a session:
- Its autopilot or plan mode is restored
- The working directory returns to where you left off (useful after `/worktree` switches)
- Long session transcripts load progressively, so even very large histories come back quickly

## Controlling Agent Permissions

The `/permissions` command lets you switch between approval modes at any time during a session — no need to restart.

### Approval Modes

| Mode | Behaviour |
|------|-----------|
| **interactive** | The agent asks for approval before each tool call |
| **plan** | The agent creates a plan and waits for approval before executing |
| **autopilot** | The agent works autonomously; you intervene when needed |

### Using `/permissions`

Switch modes mid-session:

```
/permissions
```

This opens an interactive picker. Choose the mode that suits the current task:

- Use **interactive** when you want fine-grained control or are working in a sensitive area
- Use **plan** when you want to review the overall approach before the agent starts making changes
- Use **autopilot** when you trust the task and want maximum throughput

> **Note**: The auto safety-judge model (previously user-configurable via `/allow-all`) is now selected automatically by Copilot. You no longer need to configure it manually.

### Autopilot Behaviour After Task Completion

By default, autopilot mode stays active after `task_complete`. If you prefer to return to interactive mode after each task, set `stayInAutopilot` to `false` in your settings:

```json
{
  "stayInAutopilot": false
}
```

## Predicting Session AI-Credit Limits

If your plan has an AI-credit budget, you can ask Copilot to predict an appropriate limit for a session based on similar past sessions:

```
/limits predict
```

This command analyses the current session context and similar historical sessions, then suggests a credit limit that would cover the expected work without unnecessary overage. This is useful for:

- Setting budget guardrails before starting a large refactor
- Estimating cost before running an autopilot task overnight
- Helping team leads govern AI spending across multiple developers

## Browser-Based Login

`copilot login` now defaults to a **browser-based (web) OAuth flow** on local interactive terminals. Instead of copying a device code, your browser opens automatically and completes authentication.

```bash
copilot login
```

On headless or remote terminals, the device-code flow remains the default. You can force either flow:

```bash
copilot login --web-flow      # force browser-based login
copilot login --device-code   # force device-code login
```

You can also choose your preferred login method from the interactive `/login` command inside a session.

## Queue Management

Copilot CLI includes a **directable queue manager** that lets you manage messages waiting to be sent to the agent.

When you have queued messages, you can:

- **Reorder** messages before the agent processes them
- **Edit** a message before it is sent
- **Remove** a message you no longer want
- **Repeat** a previous message
- **Send immediately** by bypassing the queue position

Press `Ctrl+C` to remove your newest queued message. The queue count in the footer shows how many messages are pending.

## Further Reading

- [GitHub Copilot CLI releases](https://github.com/github/copilot-cli/releases) — full release notes for every version
- [Copilot Configuration Basics](../copilot-configuration-basics/) — model pinning, sandbox settings, and repository configuration
- [Agents and Subagents](../agents-and-subagents/) — understanding the agent and subagent model

---
