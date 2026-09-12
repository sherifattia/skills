# Design

Read this reference when defining module seams, injecting dependencies, or choosing error contracts.

## Deep modules

Build a narrow public surface that hides substantial complexity and speaks in domain terms.

- Export the capability and the types callers need, not internal helpers, schemas, transport types, or third-party clients.
- Colocate a single-owner type with the code that owns it. Move a shared type only to the smallest boundary used by its actual consumers; avoid generic `types.ts` dumping grounds.
- Derive a type from another value or type when they are one source of truth and should evolve together. Define a separate consumer-owned shape when coupling would cross responsibilities.
- Prefer `interface extends` over intersections when intentionally composing large object hierarchies; named interfaces are easier for TypeScript to cache and display. Keep `type` as the default for closed records and unions.
- Keep implementation and tests close to the capability. Treat imports and exports, not folders, as the architectural boundary.
- Prefer plain functions and data. Use a class when identity, mutable state, or lifecycle is essential rather than as a default container.
- Avoid pass-through layers. Require each abstraction to hide a decision, enforce an invariant, translate a boundary, or enable a real substitution.
- Keep policy separate from mechanics. Do not make a retry transport decide which business failures are retryable unless that policy is its responsibility.

## Explicit dependencies

Pass the smallest capability required by the consumer. Define the dependency type near the consumer instead of forcing it to depend on a large provider interface.

Write callback and dependency contracts as function properties, such as `run: (input: Input) => Output`, rather than method signatures such as `run(input: Input): Output`. Method signatures are bivariant and can accept an unsafely narrow parameter type. Use method syntax only when that behavior is deliberate.

```ts
type Clock = () => Date;

type JobStore = {
  readonly save: (job: Job) => Promise<void>;
};

type JobServiceDependencies = {
  readonly clock: Clock;
  readonly store: JobStore;
  readonly createId: () => JobId;
};

type JobService = {
  readonly create: (input: CreateJobInput) => Promise<CreateJobResult>;
};

export function createJobService(deps: JobServiceDependencies): JobService {
  return {
    async create(input: CreateJobInput): Promise<CreateJobResult> {
      const job = makeJob(input, deps.createId(), deps.clock());
      await deps.store.save(job);
      return { tag: "created", job };
    },
  };
}
```

Make time, IDs, and I/O controllable without patching modules or relying on global state. Do not add a dependency injection container unless lifecycle wiring proves the need.

Declare return types for exported and nontrivial top-level functions so their contracts remain stable and legible. Preserve inference for small local helpers, callbacks, and JSX components where an annotation adds noise or widens a more precise inferred type.

## Errors

- Use discriminated results for expected alternatives such as rejected input, not-found, or conflict when the caller is expected to react.
- Throw for broken invariants, unavailable infrastructure, and failures that cannot be handled locally.
- Translate third-party failures once at the boundary. Preserve the original cause when it helps diagnosis without exposing library-owned error types as the public contract.
