# Troubleshooting and migration

| Symptom | Likely cause | Check or remedy |
| --- | --- | --- |
| Search returns no known records | Default project/team scope or wrong vault | Pass `scopes: ["user"]`; compare `ECC_MEMORY_USER_ROOT` in each client's server environment |
| User-scope request is denied | User scope is not enabled | Set `ECC_MEMORY_ALLOW_USER_SCOPE=1`, then restart the affected client/process |
| Vault-root symlink error | Vault or a vault directory is symlinked | Use a real directory plus the root environment override |
| `.ecc` appears after relocation | A project-scope operation was run | Inspect its contents before touching it; user relocation does not change project/team defaults |
| CLI works but GUI MCP fails | Missing or temporary executable path | Use resolved absolute Node and server-script paths; verify both still exist |
| Client sees old memory location | Existing MCP subprocess has old environment | Restart/reconnect using the client's supported lifecycle controls and test again |
| Tool is not found | Client-specific tool prefix or discovery mechanism | Discover exposed `shared-memory` tool names instead of copying another client's prefix |
| Skill loads the wrong procedure | Duplicate skill ID or source precedence | Inspect the loaded base directory and compare all copies; avoid blind overwrites |
| Handoff pointer is unusable | File moved/deleted or outside allowed workspace | Verify the absolute path and request appropriate access; do not invent continuation state |
| Doctor/search reports invalid files or truncation | Incomplete or malformed vault scan | Report the diagnostic limitation; investigate before declaring that no memory exists |

## Relocating an existing vault

1. Confirm the old vault root, contents, and which clients actually use it. The default in ECC 2.2.1 is `~/.ecc/memory`, but an override may already be active.
2. Stop writes from participating agents and make a backup outside this repository.
3. Copy the user records into a real `~/.shared-memory/memory` directory, preserving layout and stable IDs. If the destination already has records, reconcile conflicts rather than overwriting it wholesale.
4. Set `ECC_MEMORY_USER_ROOT` to that absolute directory in **every** participating client's MCP environment and in any deliberate CLI workflow.
5. Restart affected processes. Run `ecc memory doctor --scope user` with the same root override, then search/read a known ID through each client.
6. Retain the old vault/backup until verification is complete. Archive only the confirmed obsolete data; do not recursively delete `.ecc` merely because it reappears.

Do not replace the old vault with a symlink. Do not assume a root override in one terminal's environment changes GUI clients or already-running MCP servers.

## Verification checklist

- [ ] All clients point to the same real user vault.
- [ ] Each client has its own correct harness identity and user-scope opt-in.
- [ ] Absolute executable and runtime-script paths exist after upgrades.
- [ ] Each client can search and read the same known non-sensitive memory ID.
- [ ] Doctor diagnostics are understood, including skipped symlinks and incomplete scans.
- [ ] Skills resolve to the intended canonical files, with references available.
- [ ] Unmigrated skill copies are identified rather than silently discarded.
- [ ] Native auto memory/session stores are not mistaken for shared-vault records.
- [ ] Public repository changes contain templates only, not live configurations or personal data.

## Publishing technique updates

Keep this repository focused on reviewed, portable techniques. Document versions and verification limits, use placeholders for personal paths, and inspect the diff before publishing. Keep local handoffs, actual vault records, and machine-specific credentials out of commits. Commit/push only when authorized; the recorded publish convention for this repository uses its SSH remote.
