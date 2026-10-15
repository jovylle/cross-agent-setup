# Shared skills

## One canonical procedure

Keep reusable procedures under:

```text
~/.agents/skills/
└── example-workflow/
    ├── SKILL.md
    └── references/
        └── verification.md
```

Use a unique lowercase kebab-case directory name and matching frontmatter name. A portable skill can start with:

```markdown
---
name: example-workflow
description: Verify a release artifact when preparing or checking a release.
---

# Example workflow

Read references/verification.md when verifying an artifact.
Check the active project's release instructions before running commands.
```

Write descriptions that say **when to load the skill**, not just what it contains. Keep the main file focused and load supporting references only when relevant. Relative paths resolve from the skill's own directory, not necessarily the repository working directory.

## Discovery is client-specific

OpenCode V2 natively discovers `~/.agents/skills/` and also supports `~/.config/opencode/skills/` and `~/.claude/skills/`. Prefer native discovery when it already works; redundant links can introduce duplicate IDs and precedence surprises. Consult its [skills documentation](https://opencode.ai/v2/docs/skills) for the active source order.

Where a client needs its own discovery directory, link the **whole skill directory**, so references and scripts stay beside `SKILL.md`. These user-level directories were used for the recorded local setup:

| Client | Agent-specific discovery directory |
| --- | --- |
| Claude Code | `~/.claude/skills/` |
| Copilot CLI | `~/.copilot/skills/` |
| Kiro | `~/.kiro/skills/` |
| Codex | `~/.codex/skills/` |
| OpenCode | `~/.config/opencode/skills/`, when a link is actually needed |

Check the installed client's support and confirm that it advertises and loads the skill. A filesystem link alone is not proof of discovery or memory connectivity.

For example, **after confirming the destination does not already exist**:

```sh
mkdir -p "$HOME/.claude/skills"
ln -s "$HOME/.agents/skills/example-workflow" "$HOME/.claude/skills/example-workflow"
```

Do not use `ln -sf` to overwrite an unfamiliar skill. Check for dangling links as well as regular files. Windows setups may need supported native discovery or directory junctions rather than this POSIX command.

## Safe consolidation

1. Inventory existing copies and symlinks in every client directory.
2. Compare bodies, references, and scripts; different copies may contain valuable user edits.
3. Choose and reconcile a canonical copy. Back up existing material before moving it.
4. Rewrite harness-specific tool names, hardcoded personal paths, and assumed working directories where portability requires it. Preserve real safety and authorization boundaries.
5. Link clients that need it without overwriting unrelated files.
6. Confirm which definition each client actually loads and that relative references still work.

Migration can be partial. The recorded setup had a canonical `opencode-worker` skill and links for multiple clients, while other skills still had independent copies. Never treat the convention as proof that consolidation is complete.

## Keep skills and memory distinct

- Skills hold stable procedures and task-specific reference material.
- Memory holds evolving facts, preferences, decisions, and lessons.
- A skill can direct the agent to search user memory with a focused query, but should not copy the entire vault into its references.
- Existing `notes` or support-reference skills are not automatically mirrored into shared memory.
- Skills may be symlinked; the ECC memory vault may not. These are different storage mechanisms with different constraints.
