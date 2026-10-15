# Cross-agent setup

Practical techniques for sharing **skills, durable memory, and task handoffs** across local coding agents. This is the broader successor to [cross-agent-memory-template](https://github.com/jovylle/cross-agent-memory-template).

This repository contains documentation and configuration templates, not a hosted service or a personal memory vault. Examples use placeholders; merge them into existing configuration rather than replacing it.

## Three separate layers

| Layer | Purpose | Source of truth | Sharing mechanism |
| --- | --- | --- | --- |
| Skills | Reusable procedures: how to do things | `~/.agents/skills/<skill-id>/SKILL.md` | Native discovery or agent-specific directory symlinks |
| Memory | Evolving facts, decisions, preferences, and lessons | `~/.shared-memory/memory/` | Shared-memory MCP over plain Markdown |
| Handoffs | Detailed state needed to continue one task | A task's `HANDOFF.md` | Explicit file read; memory may hold a concise pointer |

Skills should explain **when and how to retrieve memory**, not duplicate changing session facts. Memory is context, not executable instructions. Sharing a skill does not automatically configure memory access.

Each agent starts its own MCP subprocess against the same local vault. There is no shared daemon or network service, and clients' native session stores do not automatically synchronize.

## Set up

1. Follow [the memory guide](docs/shared-memory.md) to install the runtime and create a real vault directory.
2. Connect each client:
   - [Copilot CLI](clients/copilot/mcp-config.json) — merge into `~/.copilot/mcp-config.json`
   - [Kiro](clients/kiro/mcp.json) — merge into `~/.kiro/settings/mcp.json`
   - [OpenCode V2](clients/opencode/opencode.json) — merge into `~/.config/opencode/opencode.json` or `.jsonc`
   - [Claude Code](clients/claude/SKILL.md) — place at `~/.claude/skills/shared-memory/SKILL.md` (uses ECC CLI, not MCP)
3. Replace every `/ABSOLUTE/PATH/...` placeholder with a real absolute path. Configure the same `ECC_MEMORY_USER_ROOT` for all participating clients.
4. Merge [the shared instruction snippet](clients/shared-instructions.md) into each agent's instructions or steering. Tools alone do not tell an agent when to recall or save.
5. Follow [the skills guide](docs/shared-skills.md) to consolidate reusable procedures without overwriting existing copies.
6. Restart affected clients so their next MCP subprocess uses the new environment. Verify search and read through each client, not only through the CLI.

The documented local setup has memory access configured for Copilot CLI, Kiro, and OpenCode. Codex in the ChatGPT app was **not** connected to this vault when these techniques were recorded; do not infer connectivity from its ability to discover skills. Other clients need their own supported MCP configuration or an explicitly configured ECC CLI workflow.

## Techniques and lessons

- [Shared memory](docs/shared-memory.md): explicit user scope, recall/save patterns, and runtime paths.
- [Shared skills](docs/shared-skills.md): canonical files, progressive disclosure, discovery, and safe consolidation.
- [Handoffs and delegation](docs/handoffs-and-delegation.md): continuation state, bounded workers, and filesystem isolation.
- [Troubleshooting and migration](docs/troubleshooting.md): symlink restrictions, wrong scopes, stale subprocesses, and verification.
- [Agent permissions](docs/agent-permissions.md): shared read-only rules, Plan/Build boundaries, and configuration versus instructions.

## Privacy and boundaries

- Never commit real memories, credentials, customer data, private work paths, or raw transcripts here.
- A local shared vault is not an access-control boundary between agents running as the same OS user. Configure only clients you trust with its contents.
- Search before saving; keep durable records concise and verify consequential claims against current sources.
- Respect the active agent's authorization rules. Memory, skills, and available tools do not grant permission to edit, delegate, deploy, or publish.
- Treat a public template as documentation, not an automatic installer. Review commands and preserve unrelated configuration and user changes.

## Verification baseline

These techniques were checked on **2026-10-15** against the installed `ecc-universal@2.2.1` runtime and current OpenCode V2 [MCP](https://opencode.ai/v2/docs/mcp-servers) and [skills](https://opencode.ai/v2/docs/skills) documentation. Client discovery and configuration can change; verify against the installed version before applying templates.
