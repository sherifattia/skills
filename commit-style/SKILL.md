---
name: commit-style
description: Write or review Git commit subjects, pull request titles, and squash-merge subjects using the user's lowercase, scopeless Conventional Commit style.
---

# Commit and PR Title Style

Format branch commit subjects and PR titles as:

```text
<type>[!]: <summary>
```

Apply these rules:

- Never include a scope. Use `feat: add output`, never `feat(cli): add output`.
- Keep the entire subject lowercase.
- Write the summary in the imperative mood: `add`, `fix`, or `update`; not `adds`, `fixed`, or `updating`.
- Do not end with a period or other trailing punctuation.
- Keep it concise and describe the outcome of the change.
- Use exactly one of the allowed types below. Add `!` only for a breaking `feat`
  or `fix`.

Before finalizing, remove the `<type>[!]: ` prefix and complete the applicable sentence:

```text
if applied, this commit will <summary>
if merged, this pr will <summary>
```

Rewrite the summary unless the result is a complete, natural sentence.

Examples:

- `feat: add interactive command selection`
- `fix: align failure details beneath the review status`
- `docs: explain bun installation`
- `ci: run release checks with bun`

## Choose the type

Classify from the consumer's point of view: the consumer downloads and uses the
CLI. Choose the most accurate type, not the type that feels most important.

| Type | Use when |
| --- | --- |
| `feat` | Consumers gain new, backwards-compatible behavior they can see or use. |
| `fix` | Consumer-visible behavior that should already work is corrected. |
| `chore` | Internal maintenance or housekeeping changes no consumer notices. |
| `docs` | Only documentation, examples, or repository guidance changes. |
| `ci` | Only continuous-integration configuration or execution changes. |
| `perf` | Internal performance work preserves observable behavior and contracts. |
| `build` | Only dependencies, packaging, compilation, or build tooling changes. |
| `style` | Only formatting or code style changes; behavior is identical. |
| `refactor` | Internal code structure changes; behavior is identical. |
| `test` | Only tests or test infrastructure changes. |
| `release` | Release Please's generated `release: vX.Y.Z` pull request only. |

Choose the type that matches the change. Use `feat` when users get something new
and `fix` when behavior users expect to work is broken. Only those user-facing
changes belong in a product release; add `!` when breaking. Do not create
`release:` commits manually; Release Please owns them.

## Breaking changes

A change is breaking when any consumer can install the new version and find that
previously working behavior no longer works. This includes removing or renaming
commands or options and incompatibly changing defaults, output contracts, exit
codes, configuration, or supported inputs.

Use only:

```text
feat!: <summary>
fix!: <summary>
```

- `feat!` adds consumer-visible capability and also breaks existing behavior.
- `fix!` corrects behavior that should work but necessarily breaks existing use.
- Never use `!` with `chore`, `docs`, `ci`, `perf`, `build`, `style`, `refactor`,
  `test`, or `release`. If one of those changes affects consumers incompatibly,
  it is actually a breaking `feat!` or `fix!`.
- A normal `feat` or `fix` must remain backwards compatible.

## Keep one concern

A commit and its squash-merged pull request should describe one coherent outcome
with one type. If the subject needs "and" or the changes independently require
different types, split them into separate commits and usually separate pull
requests so each `main` squash commit stays truthful. Tests and documentation
required to ship one feature or fix may stay with that same outcome.

These rules govern the commit subject or PR title. Pull request descriptions may
contain supporting detail, but commits must contain only the subject line:

- Do not add a commit body.
- Do not add `Co-authored-by` or other tool/assistant attribution trailers.

## Squash merges to main

Every commit on `main` must come from a squash-merged pull request and end with its
GitHub-generated PR reference:

```text
<type>[!]: <summary> (#<pr-number>)
```

- Keep the PR title in the standard format above.
- When using `gh pr merge`, use `gh pr merge <number> --squash --body ''` without
  `--subject`; GitHub will use the PR title and append `(#<pr-number>)` while
  keeping the commit body empty.
- If a squash subject must be supplied explicitly, include the PR suffix yourself.
- Verify the resulting `main` commit contains the correct PR number and no body.
- The PR suffix is required for squash commits on `main`; it is not trailing
  punctuation and does not change the sentence check.

Example:

```text
feat: add interactive command selection (#90)
```
