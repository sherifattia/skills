---
name: manage-issues
description: Refine, deduplicate, split, create, and move GitHub issues for this repository. Use when triaging the code-review-agent project or preparing tickets for implementation agents.
---

# Manage issues

Inspect the issue, its native parent/sub-issues, linked PRs, and the relevant code and tests before editing. Apply requested issue and project changes directly; do not ask for a second confirmation. Do not change implementation code during issue cleanup unless separately requested.

## Write actionable tickets

- Preserve useful existing content. Otherwise state the problem or current state, requested outcome, verifiable acceptance criteria, constraints, touch points, and relevant links.
- Call out unresolved product or contract decisions instead of inventing them.
- Keep one coherent implementation concern per issue. Target work one implementation agent can complete within roughly 150K tokens of context; split broader work into native sub-issues.
- Epics are parent issues and must have sub-issues. Do not place an epic itself in Ready as if it were an implementation task.
- When issues overlap, keep the clearest normalized issue, merge unique requirements into it, and close the redundant issue with links and a concise disposition.

## Set metadata and project state

- Every issue has exactly one GitHub issue type: `Feature` for new user-visible capability, `Bug` for behavior that should already work, or `Task` for internal, research, documentation, or maintenance work.
- Add `breaking change` only when the planned result incompatibly changes consumer behavior or a supported contract.
- Add `draft` when decisions, acceptance criteria, ownership, or decomposition are incomplete. Draft issues stay in Backlog and are not ready for an implementation agent.
- Move a non-draft issue to Ready only when its decisions are resolved, scope is bounded, dependencies are clear, and acceptance is verifiable.
- When implementation is proven, link the implementing PRs, record the delivered outcome, close the issue, and move it to Done.

After mutations, re-read the issues and project items and report their resulting types, labels, relationships, statuses, and URLs.

## Coordinate independent delivery

- Give each implementation issue one branch, one worktree, and one writing owner. Record all three in the issue before work starts; fixes continue on that same branch and worktree.
- Start an issue only when its blocked-by relationships are satisfied and its accepted interfaces are available on the intended base. Parent features coordinate children and do not own implementation branches.
- Parallelize sibling issues only when their dependencies are complete and their write surfaces are independent. A worktree isolates files; it does not resolve an overlapping contract.
- Keep the durable handoff compact: `Run`, `branch/worktree`, `base SHA`, `PR/head`, `verification`, `checks`, and `next`. Update these fields in place instead of appending a session narrative.
- Implementation and verification use fresh contexts. Verification starts from a clean checkout, confirms the full PR head SHA, and records its verdict against that SHA.
- TypeScript implementation/review issues require the `writing-modern-typescript` repository skill. If a child adds public or domain types, its acceptance requires a fresh-context exact-head type-design review covering invariant expression, constructibility, invalid-state exclusion, usefulness, and seam leakage.
- A fix moves the review target. Recheck the corrected behavior and affected integration surface at the new exact head before treating the prior verdict as current.
- Use roughly 150K tokens as the upper bound for one implementation owner, including repository reading, coding, tests, fixes, evidence, and handoff. Split work at an independently verifiable seam when that bound is not realistic.
