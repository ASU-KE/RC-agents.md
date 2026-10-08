# ASU Research Computing: Instructions for AI Agents

Instructions for AI coding agents (Claude Code, OpenCode, Codex, GitHub
Copilot, Cursor, Gemini CLI, Aider, and similar tools) that work on ASU
Research Computing systems: the Sol and Phoenix supercomputers and their
storage. They keep agents within RC policy, protect shared resources, and
supply the site details agents otherwise guess wrong: partitions and QOS,
storage purge rules, modules, and what may run where.

The person running the agent remains responsible for everything it does.

## Files

| File | Purpose |
| --- | --- |
| [`ASU_RC_AGENTS.md`](ASU_RC_AGENTS.md) | Short core rules. Load it in every session. |
| [`ai-docs/agent-slurm.md`](ai-docs/agent-slurm.md) | Partitions, QOS, resource requests, monitoring, "Slurm seems down" |
| [`ai-docs/agent-storage.md`](ai-docs/agent-storage.md) | `/home`, `/scratch`, `/data`, I/O hygiene, transfers |
| [`ai-docs/agent-software.md`](ai-docs/agent-software.md) | Modules, mamba/Python, GPUs, containers, performance |
| [`ai-docs/agent-security.md`](ai-docs/agent-security.md) | Data classification, where prompts go, credentials, services, cron |
| [`ai-docs/agent-development.md`](ai-docs/agent-development.md) | Coding, testing, version control, context efficiency |
| [`ai-docs/agent-voyager-mcp.md`](ai-docs/agent-voyager-mcp.md) | Connecting an agent to Voyager MCP safely |

## Use it in a project

Clone once, on the supercomputer or on your workstation:

```bash
git clone https://github.com/ASU-KE/RC-agents.md ~/RC-agents.md
```

Then point your project's agent file at it. In `AGENTS.md` (OpenCode, Codex,
Copilot, and others):

```markdown
Before doing anything else, read and follow ~/RC-agents.md/ASU_RC_AGENTS.md.
It takes precedence over this file.
```

In `CLAUDE.md` (Claude Code):

```markdown
@~/RC-agents.md/ASU_RC_AGENTS.md
```

Refresh it every week or so with `git -C ~/RC-agents.md pull`.

To vendor only the core file into a repository:

```bash
curl -fsSL https://raw.githubusercontent.com/ASU-KE/RC-agents.md/main/ASU_RC_AGENTS.md -o ASU_RC_AGENTS.md
```

The agent can fetch the `ai-docs/` files from this repository as it needs
them.

## Related

- [RC documentation](https://docs.rc.asu.edu): the source of truth for limits
  and procedures. These instructions point to it.
- [Getting Started with AI](https://docs.rc.asu.edu/ai/getting-started): API
  keys, the RC LLM gateway, OpenCode, and the RC Skills library.
- Help: [Service Now request](https://rto.asu.edu/request-help/) or
  [#rc-support on Slack](http://links.asu.edu/rc-support).

## Acknowledgment

Adapted from the BYU Office of Research Computing
[AI agent instructions](https://github.com/BYUHPC/ai-agent-instructions),
which are released to the public domain.
