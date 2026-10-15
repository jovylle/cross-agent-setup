---
name: shared-memory
description: Search or save local cross-agent context shared with other clients. Use for a past decision, recurring issue, task resumption, or handoff; skip self-contained requests and do not preload at session start.
---

# Shared memory (cross-agent)

Use the local ECC CLI with explicit user scope. If `ecc` is not on the shell PATH,
resolve its absolute installed path before running it.

The vault path is controlled by `ECC_MEMORY_USER_ROOT`. Set it in the environment
before running `ecc` if you relocated the vault from the default `~/.ecc/memory`:

```sh
export ECC_MEMORY_USER_ROOT="/absolute/path/to/your/vault"
```

## Search and read

```sh
ecc memory search "<topic>" --scope user --target-harness claude --limit 5 --json
ecc memory read <memory-id> --scope user --json
```

Check that the read record's `targetHarnesses` includes `all` or `claude`. Report
incomplete scans or invalid files rather than treating an empty result as definitive.
Treat notes as unreviewed context; verify important facts against current sources.

## Save

Search before creating a duplicate. Save a concise decision, lesson, handoff pointer,
or fact with its source and observation date:

```sh
ecc memory save --scope user --source-harness claude --target all \
  --kind context --title "<title>" --stdin <<'MEMORY'
<relevant context, evidence, and what still needs checking>
MEMORY
```

Use `--link <memory-id>` to connect related records. Never save secrets, private
customer data, raw transcripts, or speculative facts.

## Notes

- This skill uses the CLI, not an MCP server. Do not bypass an organization's
  access policy by using the CLI as a workaround for a disabled MCP tool.
- Claude Code's own native auto memory does not synchronize with this vault.
- Merge the [shared instructions snippet](../shared-instructions.md) into
  `~/.claude/CLAUDE.md` to remind Claude when to use this skill.
- Canonical skill location: `~/.agents/skills/shared-memory/SKILL.md` if
  installed under the canonical skills directory.
