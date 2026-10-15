---
name: opencode-worker
description: Delegate a bounded task to OpenCode as a noninteractive coding worker from another agent or script, optionally selecting a specific model and reasoning variant. Use when the user says "OpenCode Worker", "opencode worker", "call OpenCode", or asks another agent to launch an OpenCode process for implementation, review, or investigation. Not for general OpenCode configuration or unrelated background workers.
---

# OpenCode Worker

"OpenCode Worker" is the user's name for delegating a task through `opencode run`. It is a workflow, not a separate executable or built-in agent type.

## Prepare

1. Confirm delegation is authorized by the user and permitted by the parent agent's instructions. Loading this skill alone does not authorize delegation.
2. Set the subprocess working directory to the target repository or worktree. Check its instructions and existing changes before authorizing edits.
3. Define the task, relevant paths, acceptance criteria, allowed edits, verification commands, and boundaries. The worker does not inherit the parent conversation; provide the context it needs without secrets.
4. Check `opencode run --help` against the installed version before relying on flags. These examples were verified against the local CLI on 2026-10-15 and follow OpenCode V2.
5. Resolve the requested model using the available OpenCode model-discovery tool. Use the exact `provider/model#variant` reference; do not invent IDs, silently substitute another model, or treat the example below as the user's permanent default. The provider must already be authenticated.

## Launch

Use the parent shell/process tool's working-directory argument rather than interpolating a repository path into a shell command. Pass arguments as an array when the process API supports it, avoiding shell interpolation of task text.

```sh
opencode run \
  --format json \
  --model 'github-copilot/gpt-6.1-sol#high' \
  --title 'Review current changes' \
  'Review the current changes. Do not edit files or create commits. Report actionable findings with file paths and severity, and list any checks you ran.'
```

The example model and `high` variant were available locally when this skill was created; verify availability at invocation time. If no model was requested, an explicit verified model makes the run reproducible, or omit `--model` to use configured defaults and disclose that choice.

For implementation, make the boundaries explicit, for example:

```text
Implement <bounded change> in <paths>. Preserve unrelated changes.
Run <relevant verification>. Do not commit, push, deploy, or launch more workers.
Return a summary, changed paths, verification results, and remaining blockers.
```

The default uses OpenCode's shared background service. `--standalone` uses a private server, but is not a filesystem sandbox or a separate worktree.

## Permissions and parallelism

- Leave `--auto` off by default. It auto-approves permissions not explicitly denied and is not a read-only mode. Use it only when unattended execution and the corresponding access are authorized. Otherwise surface permission blockers to the user instead of silently broadening access.
- A "do not edit" prompt expresses intent, not enforced isolation. For strict read-only work, use appropriately restricted permissions or an isolated environment.
- Give concurrently editing workers separate worktrees. A separate session or private server does not isolate repository files. Use the worktree skill when available.
- Do not let delegated workers recursively delegate unless the user explicitly requested that topology.

## Capture and continue

- Capture stdout, stderr, and process exit status. `--format json` is machine-oriented output; inspect its actual event structure rather than assuming a single final JSON object or inventing a parser schema.
- Apply a task-appropriate timeout in the parent process, retain partial output, and report failures. Do not assume terminating the client cancels server-side work; verify the session state before retrying an interrupted task.
- Retain the returned session ID. Resume that explicit ID for follow-ups; avoid `--continue` in concurrent automation because "last session" can be ambiguous.

```sh
opencode run \
  --session '<returned-session-id>' \
  --format json \
  --model 'github-copilot/gpt-6.1-sol#high' \
  'Address the previous findings within the same scope. Do not commit.'
```

Use `--fork` with the session when an independent continuation is intended. Do not retry failed editing tasks blindly: inspect the diff and session first to avoid duplicate or conflicting work.

## Verify and report

Treat the worker's answer as evidence to check, not proof of success. Inspect relevant diffs and verification output. Report the selected model, session ID when available, outcome, changed paths, checks, and blockers. Distinguish a successful process exit from a successfully completed task.

For occasional delegation, prefer this CLI workflow. For a persistent orchestrator needing sessions and event streams, consult the OpenCode skill and current V2 API/client documentation instead of building brittle terminal-output scraping.

## Installation

The canonical skill lives at `~/.agents/skills/opencode-worker/SKILL.md`. Symlink it
into each client's discovery directory to share it across agents:

```sh
SKILL=opencode-worker
ln -s ~/.agents/skills/$SKILL ~/.claude/skills/$SKILL
ln -s ~/.agents/skills/$SKILL ~/.copilot/skills/$SKILL
ln -s ~/.agents/skills/$SKILL ~/.kiro/skills/$SKILL
ln -s ~/.agents/skills/$SKILL ~/.config/opencode/skills/$SKILL
ln -s ~/.agents/skills/$SKILL ~/.codex/skills/$SKILL
```

See the [shared skills guide](../../docs/shared-skills.md) for the full convention.

## References and shared recall

- CLI: https://opencode.ai/v2/docs/cli
- Programmatic client: https://opencode.ai/v2/docs/build/client
- Search shared memory with `scopes: ["user"]` and query `OpenCode Worker` to recall the naming decision and skill pointer. Memory is context, not permission to execute.
