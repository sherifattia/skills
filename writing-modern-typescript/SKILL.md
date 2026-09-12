---
name: writing-modern-typescript
description: Write and review TypeScript with strong types, deep modules, explicit boundaries, and testable design. Use for code, tests, APIs, compiler configuration, or package output.
---

# Writing modern TypeScript

## Aim

Reduce caller knowledge and make invalid states difficult to represent. Prefer deep modules: small, stable interfaces that hide substantial decisions and complexity.

## Work with the repository

- Inspect the local `package.json`, lockfile, TypeScript config, module mode, runtime, lint rules, and test setup before changing code.
- Preserve established tools and conventions unless the task requires changing them. Do not add a dependency merely because it is available.
- Keep the change no more general than its demonstrated callers require.

## Shape the design

- Decode uncontrolled input from `unknown` once, then pass validated domain values inward. Never use a type assertion as validation.
- Keep protocol DTOs, persistence records, and domain models separate when their guarantees differ.
- Derive types when they represent the same concept and should change together. Define a consumer-owned shape when the concerns have different reasons to change; do not couple a module to a larger model merely to avoid repeating a few fields.
- Represent meaningful variants with discriminated unions and handle them exhaustively with `never`.
- Model absence deliberately. Use an optional property only when the key may be omitted; use `null` only when it is part of the contract; use a named union when absence changes behavior. Do not spread `?`, `undefined`, and `null` through the domain as interchangeable escape hatches.
- Infer local implementation details, but annotate public boundaries and places where widening would weaken the contract. Use `satisfies` to check conformance while retaining useful inference.
- Prefer readonly closed records and unions with `type`. Use `interface extends` for intentional object inheritance and large composed object shapes; use declaration merging only when augmentation is the contract. Remember that `readonly` is an ownership contract, not deep runtime immutability.
- Prefer erasable syntax: literal unions or `as const` objects over enums, ECMAScript modules over namespaces, and explicit class fields over constructor parameter properties.
- Brand same-shaped primitives only when mixups are plausible and costly. Create branded values only through a checked constructor or schema.

## Keep modules deep and testable

- Expose one cohesive capability rather than wrappers for every internal function. Hide parsing, orchestration, storage details, and third-party types behind project-owned vocabulary.
- Keep side effects at the edges and domain decisions in deterministic functions. Inject clocks, ID generation, I/O, and external clients explicitly; avoid import-time work, global service locators, and mutable singletons.
- Add an adapter only for a real boundary or meaningful variation. Do not manufacture interfaces for pure helpers or mirror every implementation method one-for-one.
- Keep synchronous work synchronous. For asynchronous APIs, make ownership, cancellation, concurrency, and failure behavior explicit.
- Return a closed result for expected domain outcomes when callers must branch. Throw project-owned errors for exceptional failures; do not leak library error objects through public APIs.

Read [design](references/design.md) when creating module seams, dependencies, or error contracts.

Read [async work](references/async.md) when implementing concurrency, cancellation, retries, streams, timeouts, or background tasks.

Read [testing](references/testing.md) before implementing caller-visible behavior, changing tests, fixing a regression, or assessing release confidence. Use Vitest.

## Protect type integrity

- Do not use `any` for application data or to silence an error, and do not introduce non-null assertions. Prefer `unknown`, narrowing, schema validation, or a changed data model. Rare generic type-system plumbing may require `any`; contain it, explain why `unknown` is incorrect, and prove it with type tests plus runtime tests when behavior exists.
- Use an assertion only at a narrow boundary after a real invariant has been checked but TypeScript cannot express the proof. Keep it local, explain why it is sound, and test the invariant.
- Declare return types on exported and nontrivial top-level functions. Let trivial local functions, callbacks, and JSX components infer their return types.
- Avoid type machinery that does not simplify real call sites. Prefer a small named union or overload over opaque conditional types.
- Do not silence errors with broad casts, double assertions, blanket suppressions, fake defaults, or optional chaining that changes required behavior.
- Use `@ts-expect-error` only for a described negative type test. Do not use `@ts-ignore`.

Read [runtime boundaries](references/runtime-boundaries.md) when handling API payloads, files, environment variables, database rows, messages, or other untrusted values.

## Verify the change

- Run the repository's configured typecheck, targeted tests, lint/format checks, and then the broader relevant suite. Do not claim checks that did not run.
- For compiler or runtime changes, read [compiler and runtime](references/compiler-and-runtime.md).
- For a published package or CLI, also read [package output](references/package-output.md) and verify the built artifact, not only source execution.
