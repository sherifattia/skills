# Testing

Use Vitest. Write tests as executable contracts: preserve them across internal refactors and make them fail when caller-visible behavior breaks.

## Test the contract

- Import through the supported entry point for the contract under test. Test a package contract through the package root and an internal module through the facade used by production composition, never through private helpers.
- Assert observable return values, errors, state transitions, and owned side effects. Assert an interaction only when the interaction itself is part of the contract.
- Assert the complete value when its entire shape is promised. Otherwise assert only the fields and effects the contract guarantees.
- Include meaningful variants, boundaries, invalid input, absence, dependency failure, and state transitions. Test that forbidden side effects do not happen when their absence is promised.
- Give each test one clear reason to fail and a name that describes behavior. Keep setup, action, and assertion visible without ceremonial comments.
- For new or corrected behavior, start with a focused failing test when practical and confirm that it fails for the intended reason. Make a regression test express the violated contract, not the old implementation.

Use this refactor check: if the implementation changes while the public behavior stays the same, the tests should normally stay green.

## Choose the smallest truthful scope

1. Test deterministic domain rules directly across invariants and edge cases.
2. Test a deep module through its facade with lightweight fakes for owned boundaries.
3. Test adapters against the real parser, protocol, database, filesystem, or service emulator at the narrowest useful integration level.
4. Test critical workflows end to end only where crossing boundaries is the behavior under protection.

- Prefer the smallest scope that can observe the contract, not the smallest possible unit. Do not mock away the behavior being tested.
- Reuse one contract suite against interchangeable adapters when they promise the same semantics. Give each implementation additional tests only for its distinct risks.
- Avoid repeating the same assertion at every layer. Let each layer protect a different failure mode.

## Design for testability

- Keep decisions deterministic and side effects at owned edges. Inject clocks, randomness, IDs, I/O, and external clients as small capabilities.
- Prefer a production constructor or factory seam over intercepting imports. Test the capability with injected dependencies, then smoke-test the thin default composition separately.
- Prefer real pure collaborators. Use typed hand-written fakes for slow, nondeterministic, destructive, or externally owned dependencies.
- Do not expose a private helper, add a test-only setter, or weaken encapsulation to make a test convenient. Improve the module seam or test through the facade.
- Give every test fresh mutable state. Avoid import-time work, shared singletons, order dependence, and cleanup that relies on a later test.
- Build fixtures that are valid by default with small explicit overrides. Avoid giant shared fixtures and `beforeEach` blocks that hide why the case matters.
- Treat painful setup as design feedback. A test that must understand many collaborators usually reveals a shallow module or an oversized interface.

## Use Vitest deliberately

- Import `describe`, `expect`, `test`, and `vi` explicitly from `vitest`; do not depend on test globals.
- Prefer injected fakes or `vi.fn` at a capability boundary. Use `vi.mock` only when the import itself is the unavoidable seam.
- When module mocking is necessary, use `vi.mock(import("./module.js"), factory)` for type-checked paths and factories. Remember that it is hoisted, must be top-level, and cannot replace calls between functions in the same source file.
- Restore fake timers, spies, stubbed globals, environment variables, and mutated process state. Clearing call history does not restore all altered behavior.
- Await every promise and asynchronous assertion. Use deferred promises or explicit gates for ordering and concurrency; use `expect.poll` for genuinely eventual integration behavior instead of sleeping.
- Use fake timers only when timer behavior is under test. Inject a clock for domain time and prefer real timers for ordinary asynchronous work.
- Run tests concurrently only when their state and dependencies are isolated. Never make correctness depend on file or test execution order.
- Use separate Vitest projects only for genuinely different environments or setup, such as Node and browser execution.

## Protect public types

- Keep runtime tests and type tests separate; neither substitutes for the other.
- Add `*.test-d.ts` tests with `expectTypeOf` when public inference, assignability, overload selection, narrowing, or generic behavior is part of the API.
- Use a described `@ts-expect-error` immediately above an intentionally invalid call. Pair negative tests with positive cases so a typo or unrelated diagnostic cannot create false confidence.
- Compile representative consumer fixtures for important package contracts. Test against the emitted declarations and supported consumer configurations, not only the source project.

## Measure release confidence

- Run the focused test first, then the relevant suite. For a published package or CLI, also execute the packed artifact as described in [package output](package-output.md).
- Use coverage to find untested branches and files, not as proof of correctness. Do not add low-value assertions merely to raise a percentage.
- Use property-based tests for parsers, serializers, state machines, and algebraic invariants when generated inputs can explore more useful cases than examples. Add a library only when the risk justifies it.
- Use mutation testing selectively for critical deterministic logic when passing tests may not prove that assertions detect behavioral changes.
- For a stable published API, use the repository's existing API-report tool when signature review needs to be explicit. Do not add one during unrelated work.

## Reject brittle tests

- Do not assert private call order, intermediate variables, internal helper use, or the exact number of calls unless those details are promised behavior.
- Do not snapshot large objects, logs, or rendered trees by default. Snapshot only a deliberate serialized contract and review the diff rather than blindly updating it.
- Do not replace every dependency with a module mock. Such tests often restate wiring, miss integration failures, and resist safe refactoring.
- Do not swallow errors, loosen assertions, or update expected output merely to make a changed test pass. Decide whether the contract or the implementation is wrong.
