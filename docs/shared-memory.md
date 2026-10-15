# Shared memory

## Runtime and local vault

[ECC](https://github.com/affaan-m/ECC) provides the `ecc.memory.v1` Markdown format, CLI, and MCP server. This repository does not redistribute its runtime. The recorded baseline is `ecc-universal@2.2.1`, not a claim that this is the latest release.

For a new installation, after reviewing the dependency and ensuring Node.js/npm are available:

```sh
npm install -g ecc-universal@2.2.1
export ECC_MEMORY_USER_ROOT="$HOME/.shared-memory/memory"
ecc memory init --scope user
ecc memory doctor --scope user
```

For an existing vault, use [the migration checklist](troubleshooting.md#relocating-an-existing-vault) instead of initializing a replacement and abandoning old records.

The vault is a **real directory**, grouped by memory kind. ECC refuses symlinked vault roots and vault directories. Use `ECC_MEMORY_USER_ROOT` to relocate it; do not point a vault symlink at the new directory.

Without that override, ECC 2.2.1 defaults user memory to `~/.ecc/memory`. The override changes the user vault only: project and team roots remain separate and depend on the working directory/repository.

## MCP environment

Every participating client needs these values in its MCP server environment:

| Variable | Value | Purpose |
| --- | --- | --- |
| `ECC_MEMORY_HARNESS` | `copilot`, `kiro`, `opencode`, or the client's actual identity | Source attribution and harness targeting |
| `ECC_MEMORY_ALLOW_USER_SCOPE` | `1` | Opt into user-scope MCP access |
| `ECC_MEMORY_USER_ROOT` | Absolute path to the same real vault | Shared source of truth |

JSON strings are not shell commands. Do not use `~` or `$HOME` and assume expansion: replace template paths with absolute paths. OpenCode supports its own `{env:NAME}` substitution, but that is not a universal client syntax.

Use an absolute Node executable plus the installed `ecc-universal/scripts/memory-mcp.mjs` path, as shown in [the client templates](../README.md#set-up). Determine locations from the actual installation:

```sh
command -v node
npm root -g
```

The server script is `<npm-global-root>/ecc-universal/scripts/memory-mcp.mjs`. Resolve Node's path to the installed executable rather than a temporary per-shell version-manager link. GUI clients and non-interactive shells may not inherit your interactive shell's `PATH`. Recheck configured paths after Node or package upgrades.

Each client launches its own subprocess over MCP stdio. Restart affected clients after environment changes; existing processes retain the old values.

## Focused recall

Discover the tools exposed by your client, then use these argument shapes:

```json
{"query": "release workflow", "scopes": ["user"], "limit": 5}
```

Pass that to `memory_search`. Read a relevant returned ID with `memory_read`:

```json
{"id": "mem_example_release_workflow", "scope": "user"}
```

The ID above is illustrative; use an actual returned ID. Check targeting, trust, diagnostics, and the source date. Read only relevant records and linked notes, not the whole vault. This is bounded lexical search, not a guarantee of semantic retrieval; try alternate focused terms when appropriate.

Explicit scope matters: default project/team recall can miss user memories. A project-scope write or initialization can create `.ecc/memory/project` relative to the detected project, even after the user vault was relocated.

## Durable saves

Search first. Save only a useful decision, fact, preference, lesson, runbook, or concise handoff pointer—not a transcript dump. A synthetic `memory_save` payload looks like:

```json
{
  "title": "Example release verification rule",
  "body": "Synthetic example: verify the installed artifact after a release, not only the source build. Confirm this rule against the project's current release documentation before acting.",
  "kind": "lesson",
  "scope": "user",
  "targetHarnesses": ["all"],
  "tags": ["example", "release"]
}
```

Use `links` for related existing IDs when useful. The source harness is attributed by the server configuration. Tool-created records are unreviewed and create-only; do not pretend a new save overwrites an old record. If old context is superseded, state that relationship clearly and follow the runtime's supported maintenance workflow.

Use MCP for routine reads and saves to retain validation and concurrent-safe writes. The CLI is useful for deliberate initialization, diagnostics, and explicitly configured fallback workflows—not a reason to silently edit vault files when MCP fails.

## Verify each client

1. Confirm that its server connects.
2. Search for a known non-sensitive record with `scopes: ["user"]`.
3. Read the same returned ID with `scope: "user"` from another client.
4. Run `memory_doctor` with `scopes: ["user"]`; inspect invalid files, skipped symlinks, and truncated scans.
5. Test a cross-client save/read only when a test write is authorized. A passing CLI doctor alone does not prove that every client's subprocess uses the correct environment.
