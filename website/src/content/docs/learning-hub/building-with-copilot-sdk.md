---
title: 'Building Applications with the GitHub Copilot SDK'
description: 'Learn how to use the GitHub Copilot SDK to embed agentic AI workflows in your own applications using TypeScript, Python, Go, or .NET.'
authors:
  - GitHub Copilot Learning Hub Team
lastUpdated: 2026-07-31
estimatedReadingTime: '10 minutes'
tags:
  - sdk
  - agentic
  - developers
  - copilot-sdk
relatedArticles:
  - ./understanding-mcp-servers.md
  - ./building-custom-agents.md
  - ./using-copilot-coding-agent.md
prerequisites:
  - GitHub Copilot CLI installed and authenticated
  - Familiarity with at least one supported language (TypeScript, Python, Go, or .NET)
---

The GitHub Copilot SDK (Technical Preview) lets you embed Copilot's agentic runtime directly into your own applications. Instead of building your own AI orchestration layer from scratch, you use the same production-tested engine that powers the Copilot CLI — planning, tool invocation, file edits, streaming responses, and session management — all accessible via a clean API in your language of choice.

This article explains what the SDK does, how to get started, and when to reach for it.

> **Technical Preview**: The GitHub Copilot SDK is currently in Technical Preview and may have breaking changes. It is not yet recommended for production use.

## What Is the Copilot SDK?

The SDK exposes Copilot's agent runtime as a programmable library. Where the Copilot CLI is an interactive terminal experience, the SDK gives you the same runtime as a library you can drive from code.

**Core capabilities**:
- Create and manage agent sessions programmatically
- Send prompts and receive streaming responses
- Define custom tools that the agent can invoke
- Attach files and other context to sessions
- Connect to MCP servers for external integrations
- Query available AI models at runtime

**Supported languages**:
- TypeScript / Node.js 18+
- Python 3.8+
- Go 1.21+
- .NET / C# (.NET 8.0+)

### Architecture Overview

```
Your Application
       │
  Copilot SDK
       │  JSON-RPC
  Copilot CLI (server mode)
       │
  GitHub (models, auth, tools)
```

The SDK manages the Copilot CLI process lifecycle automatically. All communication with the underlying agent runtime happens via JSON-RPC over stdio or TCP — you write normal application code and the SDK handles the plumbing.

## Prerequisites

Before using the SDK, you need:

1. **GitHub Copilot CLI** installed and authenticated:
   ```bash
   # Install the Copilot CLI
   # See: https://docs.github.com/en/copilot/how-tos/set-up/install-copilot-cli

   # Verify installation
   copilot --version
   ```

2. **A supported language runtime** for your application.

## Installation

### TypeScript / Node.js

```bash
mkdir my-copilot-app && cd my-copilot-app
npm init -y --init-type module
npm install @github/copilot-sdk tsx
```

### Python

```bash
pip install github-copilot-sdk
```

### Go

```bash
mkdir my-copilot-app && cd my-copilot-app
go mod init my-copilot-app
go get github.com/github/copilot-sdk/go
```

### .NET

```bash
dotnet new console -n MyCopilotApp && cd MyCopilotApp
dotnet add package GitHub.Copilot.SDK
```

## Quick Start

### TypeScript

```typescript
import { CopilotClient } from "@github/copilot-sdk";

const client = new CopilotClient();

try {
  await client.start();

  const session = await client.createSession({
    onPermissionRequest: async (request) => ({ approved: true }),
  });

  await session.sendAndWait({
    prompt: "List the files in the current directory",
  });
} finally {
  await client.stop();
}
```

### Python

```python
from github_copilot_sdk import CopilotClient

async def main():
    client = CopilotClient()
    await client.start()

    try:
        session = await client.create_session()
        response = await session.send_and_wait(
            prompt="List the files in the current directory"
        )
        print(response.text)
    finally:
        await client.stop()
```

## Key Patterns

### Multi-turn Conversations

The SDK maintains conversational state across multiple `sendAndWait` calls within a session:

```typescript
const session = await client.createSession({
  onPermissionRequest: approveAll,
  model: "gpt-4.1",
});

await session.sendAndWait({ prompt: "My project uses React and TypeScript" });
const response = await session.sendAndWait({
  prompt: "Suggest a folder structure for my project",
});
// The agent remembers the context from the first turn
```

### File Attachments

Attach files directly to sessions for analysis or context:

```typescript
await session.send({
  prompt: "Review this code for potential issues",
  attachments: [
    {
      type: "file",
      path: "./src/auth.ts",
      displayName: "Authentication Module",
    },
  ],
});
```

### Querying Available Models

You can programmatically list which AI models are available at runtime:

```typescript
const models = await client.getModels();
// Returns: ["gpt-4.1", "gpt-4o", "claude-sonnet-4.5", ...]
```

This is useful if your application needs to let users pick a model, or if you want to switch models based on task complexity.

### Graceful Shutdown

Always clean up the client to properly terminate the underlying CLI process:

```typescript
process.on("SIGINT", async () => {
  console.log("Shutting down...");
  await client.stop();
  process.exit(0);
});
```

## When to Use the Copilot SDK

The SDK is the right choice when you want to:

| Scenario | Why the SDK Fits |
|----------|-----------------|
| Build an internal automation tool | Embed Copilot into a CI pipeline, Slack bot, or internal dashboard |
| Create a domain-specific assistant | Wrap Copilot with your own UI and context for a specialized task |
| Prototype agentic workflows | Test agent behavior programmatically before deploying to production |
| Integrate Copilot into an existing application | Add AI-powered features without switching to a new tool |
| Script multi-step agent tasks | Orchestrate sequences of agent interactions with code |

### SDK vs Other Copilot Surfaces

| Surface | Best For | Programmable? |
|---------|----------|---------------|
| **VS Code extension** | Real-time inline assistance while coding | No (interactive) |
| **Copilot CLI** | Terminal-first development, interactive sessions | Via automation, not API |
| **Copilot app** | Parallel agents, visual workflow management | No (interactive) |
| **Copilot SDK** | Building custom applications, automation, programmatic control | Yes |
| **Agentic Workflows** | Scheduled/event-driven automation in GitHub Actions | Via markdown spec |

Use the SDK when you need full programmatic control over agent sessions from within your own application code.

## Best Practices

1. **Always clean up**: Use `try-finally` or `defer` to ensure `client.stop()` is called, even on errors.
2. **Set timeouts**: Use `sendAndWait` with a timeout for long operations to avoid hanging indefinitely.
3. **Handle errors**: Subscribe to error events for robust error handling in production scenarios.
4. **Use streaming for long responses**: Enable streaming to improve perceived performance in user-facing applications.
5. **Reuse sessions for multi-turn**: Creating sessions has overhead — reuse them for related back-and-forth conversations.
6. **Define tools clearly**: Write descriptive tool names and descriptions so the agent can decide when to invoke them.

## Installing the Copilot SDK Plugin

The community has packaged the SDK documentation and examples as an installable Copilot CLI plugin, which adds an `/copilot-sdk` skill with detailed language-specific instructions:

```bash
copilot plugin install copilot-sdk@awesome-copilot
```

Once installed, use `/copilot-sdk` in any Copilot CLI session to get hands-on guidance for building with the SDK in your chosen language.

## Further Reading

- **GitHub Copilot SDK Repository**: [github/copilot-sdk](https://github.com/github/copilot-sdk) — Source code, examples, and tutorials
- **Getting Started Tutorial**: [First App Guide](https://github.com/github/copilot-sdk/blob/main/docs/tutorials/first-app.md)
- **SDK Cookbook**: [Example applications and recipes](https://github.com/github/copilot-sdk/tree/main/cookbook)
- **MCP Servers**: [Understanding MCP Servers](../understanding-mcp-servers/) — Connect your SDK applications to external tools
- **Building Custom Agents**: [Building Custom Agents](../building-custom-agents/) — Define agent personas and behaviors
- **Agentic Workflows**: [Agentic Workflows](../agentic-workflows/) — Scheduled automation without writing application code

---
