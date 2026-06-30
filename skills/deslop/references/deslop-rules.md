# Deslop Rules

This file defines slop categories, intent-aware removal criteria, and guardrails for the
`deslop` skill. The SKILL.md workflow reads this file at deslop time.

## Slop Categories

Remove code introduced on the branch that matches these patterns **unless intent context
says it was explicitly requested or required**:

| Category | Remove when | Keep when |
| --- | --- | --- |
| **Comments** | Obvious narration (`// increment counter`), section banners, doc comments on unchanged trivial code, comments inconsistent with file tone | Explains non-obvious business logic, documents a constraint from the session, required by linter |
| **Defensive checks** | Redundant null/empty guards on trusted or already-validated paths; duplicate validation layers | Checkpoint intent mentions edge cases, user input, external I/O, or explicit error handling |
| **try/catch & error handling** | Catch-all blocks that swallow errors, nested try/catch abnormal for the file, logging-and-rethrow with no value | Intent asked for resilience, graceful degradation, or user-facing error messages |
| **Type escapes** | `as any`, `@ts-ignore`, `@ts-expect-error`, unchecked casts added only to silence the compiler | Intent acknowledges a known type gap with a documented workaround |
| **Over-abstraction** | One-off helpers, wrapper types, or indirection not used elsewhere in the file | Intent asked for reuse, testability, or matching an established pattern in the repo |
| **Scope creep** | Unrelated refactors, drive-by renames, formatting churn outside touched logic, new utilities not tied to the task | Intent explicitly included cleanup, refactor, or broader change |
| **Style drift** | Naming, spacing, or patterns inconsistent with the surrounding file and codebase | Matches local conventions or intent requested a convention change |
| **Leftover artifacts** | Debug logs, `TODO`/`FIXME` from the session, commented-out code, temporary flags | Intentional feature flags or tracked follow-ups mentioned in intent |

## Intent-Aware Removal

When checkpoint transcripts are available for a file, use them to decide **keep vs remove**:

- **Minimal scope** — If the session asked for a small or focused change, remove additions
  not required to satisfy that request.
- **Explicit requests** — If the transcript asked for comments, logging, validation, or
  error handling, do not strip those even if they look like slop in isolation.
- **Discussed tradeoffs** — If the session chose a defensive approach for a reason, keep it
  unless the reason no longer applies in the final code.
- **Rejected alternatives** — Remove code paths or abstractions the user declined during the
  session if they still appear in the diff.
- **Stated constraints** — Honor "don't change X", "keep it simple", "match existing style",
  or "no new dependencies" from the transcript when deciding removals.

When checkpoint context is **unavailable**, fall back to file-local heuristics: compare
added lines to the style and error-handling patterns of the unchanged parts of the same
file and neighboring call sites.

## Guardrails

1. **Behavior unchanged** — Do not alter runtime behavior unless removing dead code or fixing
   an obvious bug introduced alongside slop.
2. **Minimal edits** — Prefer deleting or simplifying added lines over rewriting unrelated
   code. Do not refactor for taste beyond the branch diff.
3. **Do not deslop pre-existing code** — Only touch lines introduced or materially changed
   on the branch (committed and uncommitted), unless the user asks otherwise.
4. **Preserve public API** — Do not rename exports, change signatures, or reorder public
   interfaces unless required to remove slop without behavior change.
5. **Skip generated and lock files** — Unless the user explicitly includes them.
6. **When uncertain, keep** — If intent is ambiguous and removal might drop requested work,
   leave the code and note it in the summary instead of deleting.

## What Counts as "Introduced on the Branch"

Use `git diff <base>..HEAD` plus `git diff HEAD` for uncommitted work. A line is in scope if
it appears in the diff as an addition or as part of a changed hunk. Context-only unchanged
lines in a hunk are out of scope unless they are slop left behind by a partial edit (e.g.
an orphaned comment above new code).
