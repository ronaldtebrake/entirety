---
name: deslop
description: >
  Remove AI-generated slop from branch changes using checkpoint transcript context
  to distinguish requested work from unnecessary additions. Use when the user asks
  to deslop, clean up AI slop, remove agent cruft, or tidy branch changes before merge.
---

# Deslop

Check the diff against main and remove AI-generated slop introduced on the branch — but
only slop that does **not** match the developer's stated intent from Entire checkpoints
and the active session.

## Response Format

Begin the first response to this skill invocation with the line:

`Entire Deslop:`

followed by a blank line, then the work.

- Apply the header to the **first response of the invocation only.**
- Do **not** include the header on error or early-exit responses (e.g. "not a git
  repository", "no changes found").

End with **only** a 1–3 sentence summary of what you changed. No bullet lists, no
file-by-file report, no follow-up questions unless blocked.

## Rules

1. Read `references/deslop-rules.md` before editing.
2. Only remove slop on lines introduced or changed on the branch unless the user says
   otherwise.
3. Use checkpoint intent to **protect** code the developer asked for, even if it looks
   like slop out of context.
4. Keep behavior unchanged unless removing dead code or an obvious bug bundled with slop.
5. Prefer minimal deletions and local simplifications over broad rewrites.

## Process

### 1. Verify environment

Run `entire version`. If the CLI is missing, proceed in degraded mode (file-local
heuristics only) and note that in the closing summary.

### 2. Detect base ref

Try these refs in order and use the first that exists:

```bash
git rev-parse --verify origin/HEAD 2>/dev/null
git rev-parse --verify origin/main 2>/dev/null
git rev-parse --verify origin/master 2>/dev/null
git rev-parse --verify main 2>/dev/null
git rev-parse --verify master 2>/dev/null
```

If none exist, ask the user for a base ref and stop.

### 3. Compute scope

```bash
git rev-list --count <base>..HEAD
git diff --stat <base>..HEAD
git diff --stat HEAD
```

If there are no committed or uncommitted changes relative to scope, stop and tell the user.

### 4. Gather checkpoint intent (skip in degraded mode)

```bash
git log --format='%H%x00%s%x00%b' <base>..HEAD
```

Extract `Entire-Checkpoint:` trailer values from commit bodies. Deduplicate checkpoint IDs.

For each unique checkpoint ID (up to 20):

```bash
entire explain --checkpoint <checkpoint-id> --json --no-pager
```

Parse JSON for session summaries and user prompts. Prefer summary (intent + outcome); fall
back to the latest user prompt. Truncate each checkpoint detail to 320 characters.

If `--json` fails:

```bash
entire explain --checkpoint <checkpoint-id> --no-pager
```

Do not use `--full` — it is too verbose for bulk intent mapping.

Read the active session for uncommitted intent:

```bash
entire session current --json
```

Extract `last_prompt` and `files_touched` when present.

Build an intent map: for each file in the diff, collect relevant intent from checkpoints
and the active session. Note explicit constraints ("minimal change", "add validation",
"match existing style", "no comments").

### 5. Read the diff

```bash
git diff <base>..HEAD
git diff HEAD
```

For large diffs (>5000 lines), prioritize files with the most added slop signals and files
with checkpoint intent. Read per-file diffs as needed.

Before editing a file, read enough surrounding context to match local style and
error-handling patterns.

### 6. Remove slop

For each in-scope file and hunk, apply `references/deslop-rules.md`:

**With checkpoint context:**

- Remove additions that violate stated intent or scope ("keep it minimal" → strip extras).
- Keep additions the transcript explicitly requested (validation, comments, logging).
- Remove scope creep and artifacts not discussed in the session.
- Remove defensive layers the session rejected or never mentioned for trusted paths.

**Always (with or without context):**

- Extra comments a human would not add or that clash with the file
- Abnormal try/catch or redundant guards on trusted code paths
- `any` casts and type suppressions used only to bypass errors
- Deep nesting simplifiable with early returns without behavior change
- Debug logs, commented-out code, and stale TODOs from the branch work

When intent and slop heuristics conflict, **keep the code** and mention the ambiguity
briefly in the closing summary.

### 7. Verify

After edits:

- Ensure changed files still typecheck/build if the project has an obvious check command
  (e.g. `npm test`, `go test ./...`) — run it only when fast and discoverable from
  repo scripts; do not guess a long CI pipeline.
- Re-read the diff you produced; confirm no unintended behavior changes.

### 8. Summarize

Close with 1–3 sentences total. Example:

> Removed narrating comments and a redundant null guard in `auth.ts` where the caller
> already validates input. Kept the try/catch in `fetchUser` because the checkpoint intent
> asked for graceful API error handling. No other slop matched the branch diff.

## Degraded Mode (no Entire CLI or no checkpoints)

1. Skip step 4.
2. Use file-local style and call-site patterns only.
3. When uncertain, keep rather than delete.
4. Mention in the summary that intent context was unavailable.

## Failure Modes

- **No base ref** — Ask the user to specify one; do not edit files.
- **No diff** — Report cleanly; do not edit files.
- **Diff too large (>100 files)** — Deslop the highest-signal files first (most additions,
  checkpoint-linked files). Summarize what was covered and what was skipped.
- **`entire explain` fails for a checkpoint** — Continue without that checkpoint's intent.
- **Auth required** — Tell the user to run `entire login`, then proceed in degraded mode if
  they want to continue without waiting.
