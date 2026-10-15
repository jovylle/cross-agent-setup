# Handoffs and delegation

## Continuation state belongs in a handoff

A durable memory is not a substitute for the detailed state of an unfinished task. Keep that state in a task's `HANDOFF.md`, using an existing project convention where available:

```markdown
# Task handoff

## Location
- Repository/worktree: <absolute path>
- Branch: <branch>
- Relevant instructions: <paths>

## Objective and boundaries
<Requested outcome, acceptance criteria, and prohibited actions>

## Current state
<Completed work, changed files, and unrelated pre-existing changes>

## Verification
<Commands run, actual results, and checks not yet run>

## Blockers and next steps
<Unresolved choices and the next concrete actions>
```

Keep secrets and raw transcripts out. If useful, save a concise durable decision and a pointer to the handoff's **absolute path** in shared user memory. Verify the file and current Git state when resuming; a pointer can become stale after a worktree is removed.

## Bounded cross-agent workers

The recorded setup calls delegation through `opencode run` **OpenCode Worker**. This is a reusable workflow/skill, not a separate executable or built-in agent type. Its canonical skill lives at `~/.agents/skills/opencode-worker/SKILL.md` where installed.

Before launching any worker:

1. Confirm delegation is authorized by the user and the parent harness.
2. Set the process working directory to the intended repo or worktree and inspect its instructions and existing changes.
3. Provide the objective, relevant paths, allowed edits, acceptance criteria, verification commands, and boundaries. The worker does not inherit the parent conversation.
4. Check the installed CLI's help and available model IDs. Do not invent a model reference or silently substitute one.
5. Capture output, errors, exit status, and a session ID when available. Verify the worker's diff and checks rather than trusting its final summary.

For OpenCode syntax, use its current [V2 CLI guide](https://opencode.ai/v2/docs/cli) and installed `opencode run --help`. This repository documents the technique, not a frozen model choice or automatic delegation policy.

## Isolation and permissions

- Separate sessions and private servers do **not** isolate repository files. Give concurrent editing workers separate worktrees.
- A “do not edit” prompt expresses intent, not enforced read-only isolation. Use restricted permissions or an isolated environment where strict enforcement is required.
- Do not enable unattended auto-approval unless that access is explicitly authorized.
- Prohibit recursive delegation, commit, push, or deployment unless those actions are within the request.
- On interruption, inspect partial changes and worker/session state before retrying. Terminating a client process does not necessarily cancel server-side work.
- Resume an explicit session ID; a “last session” shortcut is ambiguous during concurrent automation.
