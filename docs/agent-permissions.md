# Agent permissions

## Instructions are not permission configuration

An instruction or steering file tells an agent how to behave. The harness's permission configuration decides whether a tool is allowed, denied, or requires approval. Updating this repository or an `AGENTS.md` file does not by itself change live tool permissions.

In OpenCode V2, user-level permissions belong in `~/.config/opencode/opencode.json` or `.jsonc`. Put shared inspection rules in the top-level `permissions` array so both Plan and Build receive them. Use agent-specific rules only for genuine differences, such as Plan's edit restrictions.

This fragment illustrates the structure; merge it into existing rules rather than replacing the entire configuration:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "permissions": [
    { "action": "shell", "resource": "*", "effect": "ask" },
    { "action": "webfetch", "resource": "*", "effect": "allow" },
    { "action": "websearch", "resource": "*", "effect": "allow" },
    { "action": "glob", "resource": "*", "effect": "allow" },
    { "action": "grep", "resource": "*", "effect": "allow" },
    { "action": "shell", "resource": "git status *", "effect": "allow" },
    { "action": "shell", "resource": "git diff --no-ext-diff --no-textconv *", "effect": "allow" },
    { "action": "shell", "resource": "git log --no-ext-diff --no-textconv *", "effect": "allow" },
    { "action": "shell", "resource": "git ls-remote *", "effect": "allow" }
  ]
}
```

This is a starting fragment, not a complete security policy. Preserve protections for secrets, external directories, shell output/redirection, and non-inspection commands.

## Why an earlier allowance may still prompt

- OpenCode uses the **last matching rule**. A later broad shell guard can override an earlier command allowance.
- Agent rules are appended after shared rules. Inspect both the shared array and the active agent's overrides.
- Quoted paths, redirection, pipes, and compound commands may match additional rules or scanner-produced resources. Test actual command forms, not just an unquoted example.
- External-directory access is a separate permission check.
- “Allow always” approvals are project-scoped; an approval in one repository is not necessarily global.
- Configuration and installed versions matter. V2 uses `permissions`, `shell`, and `agents.<id>.permissions`; avoid writing new settings using legacy field names.

## Distinguish inspection from permitted metadata changes

`git fetch` does not edit working-tree files, but it **does update local Git metadata**. If it should run without prompting, authorize it explicitly rather than describing it as strictly read-only. That decision is separate from permission to merge, reset, commit, or push.

Likewise, a command named `find` is not automatically read-only: options such as `-delete`, `-exec`, and `-execdir` can mutate files or execute other programs. Prefer dedicated glob/grep/read tools where possible; do not blanket-allow `find *` just to permit directory inspection.

Permission patterns match command text, not a complete proof of safety. Keep shell allowlists narrow and verify representative commands in both Plan and Build. Plan should retain its normal prohibition on implementation edits even when inspection is allowed.

Reference: [OpenCode V2 permissions](https://opencode.ai/v2/docs/permissions), checked on 2026-10-15. This guide does not modify any live permission settings.
