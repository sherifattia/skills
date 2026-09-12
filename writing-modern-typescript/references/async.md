# Async work

Read this reference for asynchronous APIs, concurrent work, cancellation, deadlines, retries, streams, or background tasks. Keep synchronous work synchronous.

## Choose the execution shape

| Need | Prefer |
| --- | --- |
| Ordered or dependent steps | `for...of` with `await` |
| Small, known set of independent tasks | `Promise.all` |
| Every success and failure is meaningful | `Promise.allSettled` with explicit result handling |
| First successful result | `Promise.any`, then cancel work that is no longer needed |
| Large or rate-limited workload | An established limiter, queue, or worker pool |
| Large or unbounded sequence | An async iterable or stream with backpressure |

- Document whether results preserve input order and whether partial success is possible.
- Do not use an async callback with `forEach`; it neither awaits nor collects the returned promises.
- Avoid `new Promise(async ...)`. Adapt callback APIs once at the boundary, then use promises internally.
- Do not parallelize operations that depend on shared mutable state, ordering, or a previous result.

## Own every task

- Await, return, or collect every promise. A detached task needs a named owner that reports failures and participates in shutdown; `void startTask()` alone does not establish ownership.
- Do not pass a promise-returning function to a callback contract that ignores its result. Wrap it and handle failures explicitly, or change the contract.
- `Promise.all` rejects when one input rejects, but it does not cancel the remaining work. If fail-fast cancellation matters, coordinate the operations with a shared `AbortController` and ensure each operation observes its signal.
- Use a closed result when each failure is an expected caller decision. Throw or reject for exceptional failure, and translate third-party errors at the module boundary.
- Do not mark a function `async` merely to wrap a value. Keep the return type honest about whether work is asynchronous.

## Cancellation and deadlines

- Accept an `AbortSignal` in cancellable operations and propagate it through every supporting dependency. Check `signal.throwIfAborted()` before starting work and between long CPU-bound chunks.
- Treat a timeout as a cancellation request, not proof that work stopped. The operation must observe the signal and release its resources.
- If a dependency cannot observe cancellation, stop admitting new work and document that already-running work may continue.
- When the supported runtime provides them, compose caller cancellation and deadlines with `AbortSignal.any` and `AbortSignal.timeout`; preserve the abort reason instead of replacing it with a generic error.
- Register abort listeners with `{ once: true }` and remove listeners, clear timers, close handles, and release resources in `finally`.
- Use `await using` only when the repository's runtime, compiler, and resource types support asynchronous disposal. Otherwise use an explicit `try`/`finally`.

## Bound concurrency and apply backpressure

- Set concurrency from downstream capacity such as connection pools, rate limits, memory, or CPU. Batching does not by itself cap concurrent work.
- For a large collection, prefer the repository's existing limiter, queue, or pull-based worker pool. Avoid eagerly creating or enqueuing every task when memory or cancellation matters. Do not hand-roll a scheduler unless the required semantics are genuinely different.
- Consume large or unbounded data with `for await...of` or the platform's stream APIs instead of materializing the entire input. Propagate cancellation and respect the producer's backpressure contract.
- Avoid read-modify-write races across awaits. Use atomic storage operations, per-key serialization, or an owned queue when state must change consistently.

## Retry deliberately

- Retry only failures classified as transient. Do not retry validation, authorization, programmer, invariant, or caller-cancellation errors.
- Bound maximum attempts and overall elapsed time. For remote dependencies, add jittered backoff, honor server retry guidance, and stop immediately when the signal is aborted.
- Retry writes only when the operation is idempotent or protected by an idempotency key or equivalent deduplication guarantee.
- Decide whether the deadline covers the whole operation or each attempt, and expose that behavior in the module contract.

## Test async behavior

- Inject sleep, clocks, queues, and external capabilities where policy needs control. Avoid real-time sleeps and broad module mocks.
- Use deferred promises or explicit gates to prove ordering, maximum in-flight work, cancellation before and during execution, cleanup, and partial-failure behavior deterministically.
- Test that cancellation stops retries and prevents new work from starting. Test what happens to already-running work.
- Assert through the public interface and check for reported background failures; do not couple tests to incidental microtask counts.
- Enable the repository's type-aware promise lint rules, especially checks for floating and misused promises, when the lint stack supports them.
