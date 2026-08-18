---
title: 'Multi-Client Sessions with Agent Host Protocol (AHP)'
description: 'Learn how to use the Agent Host Protocol (AHP) to share Copilot sessions across multiple terminals, Codespaces, and cloud environments.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-08-18
estimatedReadingTime: '10 minutes'
tags:
  - ahp
  - sessions
  - collaboration
  - fundamentals
relatedArticles:
  - ./using-copilot-coding-agent.md
  - ./copilot-configuration-basics.md
  - ./understanding-mcp-servers.md
prerequisites:
  - Familiarity with GitHub Copilot CLI
  - Copilot CLI v1.0.79 or later
---

The Agent Host Protocol (AHP) lets you run a Copilot session on one machine and attach to it from multiple terminals, a Codespace, or a cloud environment — all simultaneously. Instead of each terminal running its own independent agent, AHP centralises sessions on a *host* and lets any number of clients attach, observe, and steer the same conversation.

This article explains how AHP works, how to start and attach to a host, and how to connect across different compute environments.

## What Is AHP?

AHP introduces a **host/client model** for Copilot sessions:

- **Host** (`copilot --ahp`): Runs `copilotd`, a local daemon that owns the sessions. Sessions live on the host and persist independently of any individual terminal.
- **Client** (any `copilot` instance): Attaches to a host and joins its sessions. Multiple clients can be attached to the same session at the same time.

When multiple clients are attached, they all see the same streaming output. You can type from any attached terminal — including steering a turn that another terminal started.

### Why Use AHP?

| Scenario | Benefit |
|----------|---------|
| Multi-monitor setup | Watch agent output on one screen while typing prompts on another |
| Pair programming | Two developers attached to the same session, each able to steer |
| Headless monitoring | One terminal runs the agent; another drops in to check progress |
| Codespace workflows | Attach from a local terminal to an agent running in a Codespace |
| Cloud agent access | Connect to a Mission Control cloud environment session |

## Getting Started

### Starting a Local AHP Host

Launch the CLI with `--ahp` to attach to (or start) a local host:

```bash
copilot --ahp
```

- If a local AHP daemon is already running, `--ahp` attaches to it.
- If no daemon is running, `--ahp` starts one in the current directory and attaches.

The **Sessions tab** (accessible with `s` or via the sidebar) lists all sessions on the host, clearly marked `AHP` so you know they live on the daemon rather than in your local process.

### Attaching a Second Terminal

Open a second terminal and run the same command:

```bash
copilot --ahp
```

Both terminals are now attached to the same host. Navigate to the Sessions tab, select a running session, and both terminals show the same output. Either terminal can send messages to steer the agent.

The session header shows the number of attached clients (e.g., `2 clients`) so you always know when someone else is also connected.

### Managing the Daemon with `/ahp` Commands

Once inside a `--ahp` session, use `/ahp` slash commands to manage hosts and sessions:

```
/ahp status                   # show identity and health of the current host
/ahp start [port]             # start a new AHP daemon in the current directory
/ahp stop <host>              # stop a named host (requires --force if you're in its session)
/ahp restart <host>           # restart a host on the same workspace it was serving
/ahp hosts                    # list all known hosts and their health
/ahp use <host>               # switch the Sessions tab to a different source
/ahp connect <url>            # add a remote host by URL
/ahp sessions                 # list sessions on the current host
/ahp attach <session-id>      # attach to a specific session
/ahp new                      # create a new session on the current host
```

> **Tip**: `/ahp start` also serves as a fix if a healthy host is refusing new sessions with a "permission denied" error. Starting a new daemon in your current directory resolves workspace-scope mismatches.

### Auto-Discovery of Local Daemons

When you run `copilot --ahp`, the CLI automatically discovers AHP daemons already running on your machine and lists them in the Sessions tab's source picker. You don't need to name or connect them explicitly — a daemon you started in another terminal appears automatically, including ones that start *after* the CLI is already open.

To disable auto-discovery:

```bash
COPILOT_AHP_DISCOVER=0 copilot --ahp
```

### Connecting Multiple Hosts

`--ahp` and `COPILOT_AHP_URL` accept a comma-separated list of host URLs, and `/ahp connect <url>` adds a host while the CLI is running:

```bash
copilot --ahp "wss://remote-host:8765,wss://another-host:8766"
```

Use `h` in the Sessions tab to switch between sources. The source picker shows each host's health beside its entry — so a host that stops responding is immediately visible rather than appearing to have no sessions.

## Connecting to Remote Environments

AHP is not limited to local daemons. You can attach to sessions running in Codespaces and Mission Control cloud environments.

### Codespaces

Forward a Codespace's `copilotd` port to your local machine with:

```
/ahp codespace <name>
```

The Codespace appears in the Sessions tab's source picker, labelled `CS`. The tunnel closes when you exit or when you run `/ahp stop <name>`.

You can use either the auto-generated Codespace name (`gh codespace list`) or the display name you gave it. If the name matches nothing, the command lists your available Codespaces.

> **Note**: The `codespace` OAuth scope is required. If it's missing, the command tells you exactly which `gh auth refresh` command to run.

### Mission Control Cloud Environments

```
/ahp cloud <environment-id>
```

This puts a Mission Control cloud environment in the Sessions tab's source picker, labelled `CLOUD`. Mission Control wakes the environment on connect — the CLI cannot start or stop it directly.

### Connection Tokens

Remote hosts can be protected with a connection token appended to the URL:

```bash
copilot --ahp "wss://my-host:8765?tkn=…"
```

The CLI redacts the query string from all output (`/ahp status`, session lists, error messages) so transcripts can be shared without leaking the token. A `401` response tells you clearly that the token is what the host requires.

## Session Behaviour

### Sessions Belong to the Host

Sessions running on an AHP host exist independently of any attached client:

- Closing a client terminal does **not** end the session.
- The agent keeps working — you can re-attach from any terminal.
- Other attached clients continue to see output without interruption.

### Steering and Queuing

While a turn is streaming, all attached clients can interact:

- **Enter** steers the prompt into the running turn.
- **Ctrl+Q** queues it for the next turn.
- **Ctrl+C** cancels the current message.

Each attached client sees the same turn stream, including messages sent by other clients.

### Session Directory

An `--ahp` session runs in the directory you started the CLI from, *if* the host's workspace covers it. If you point a daemon at a parent directory, attaching from a subdirectory runs the agent in that subdirectory rather than at the workspace root.

### Session Lifecycle Commands

`/clear` and `/new` are fully supported in AHP sessions — they replace the session on the host itself, so every attached client sees the reset and can continue interacting with the fresh session.

## Checking AHP Status

Use `/ahp status` to see:

- The daemon identity (`copilotd` version)
- Whether this CLI started the daemon or attached to one already running
- The host's health (still responding, unreachable, etc.)
- How many clients are currently attached

If the host goes unreachable, the CLI announces the loss once in the timeline (since you may not be watching the Sessions tab at that moment) and marks the host's entry in the source picker accordingly.

## Best Practices

- **Start the daemon at your project root**: The workspace scope determines which directories clients can run sessions in. Starting `copilot --ahp` at the root of a monorepo lets clients work in any subdirectory.
- **Use `/ahp status` to troubleshoot**: If sessions behave unexpectedly (wrong directory, missing skills), status tells you which daemon owns the session and whether it's healthy.
- **Protect remote hosts with tokens**: When forwarding a host over a network, use the `?tkn=` query parameter to require authentication from connecting clients.
- **Use `/ahp restart` for daemon upgrades**: After updating the CLI, restart the daemon with `/ahp restart` to pick up the new `copilotd` binary without losing your session list.

## Further Reading

- [Copilot CLI releases — v1.0.79](https://github.com/github/copilot-cli/releases/tag/v1.0.79) — full AHP feature notes
- [Using the Copilot Coding Agent](../using-copilot-coding-agent/) — remote control and cloud agent sessions
- [Copilot Configuration Basics](../copilot-configuration-basics/) — startup flags and session modes

---
