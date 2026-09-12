# Runtime boundaries

Read this reference when data enters from JSON, HTTP, environment variables, files, queues, databases, browser storage, or an untyped library.

## Boundary recipe

1. Receive the value as `unknown` or the honest transport type.
2. Validate only guarantees the source actually provides.
3. Convert transport absence and variants into explicit domain states.
4. Translate validation failures into project-owned error fields.
5. Export the domain value or a library-neutral result, not the validator's error type.

Use the validator already established by the repository. When using Zod, derive types from the schema with `z.input` and `z.output`; do not duplicate the shape by hand. Use `safeParse` when rejection is an expected result and `parse` when the API is intentionally exception-based.

## Example

```ts
import { z } from "zod";

const eventIdSchema = z.string().regex(/^[0-9a-f]{32}$/).brand<"EventId">();

const sourceEventSchema = z.object({
  id: eventIdSchema,
  kind: z.enum(["prompt", "tool", "response"]),
  parentId: eventIdSchema.nullable().optional(),
  payload: z.unknown(),
});

type SourceEvent = z.output<typeof sourceEventSchema>;

type Parent =
  | { readonly tag: "present"; readonly id: SourceEvent["id"] }
  | { readonly tag: "missing" };

type Event = {
  readonly id: SourceEvent["id"];
  readonly kind: SourceEvent["kind"];
  readonly parent: Parent;
  readonly payload: unknown;
};

type DecodeIssue = {
  readonly code: "invalid_source_event";
  readonly path: readonly string[];
  readonly message: string;
};

type DecodeResult =
  | { readonly tag: "decoded"; readonly value: Event }
  | { readonly tag: "rejected"; readonly issues: readonly DecodeIssue[] };

export function decodeEvent(input: unknown): DecodeResult {
  const parsed = sourceEventSchema.safeParse(input);

  if (!parsed.success) {
    return {
      tag: "rejected",
      issues: parsed.error.issues.map((issue) => ({
        code: "invalid_source_event",
        path: issue.path.map(String),
        message: issue.message,
      })),
    };
  }

  const { id, kind, parentId, payload } = parsed.data;
  return {
    tag: "decoded",
    value: {
      id,
      kind,
      parent: parentId === null || parentId === undefined
        ? { tag: "missing" }
        : { tag: "present", id: parentId },
      payload,
    },
  };
}

export type { DecodeResult, Event };
```

The conversion deliberately treats omitted and `null` parent IDs as the same domain state. Keep them distinct if the protocol or behavior distinguishes them. `payload` remains opaque; decode it again at the boundary of the component that owns its shape.

## Modeling rules

- Do not accept `Record<string, unknown>` merely to avoid validation; it proves only that the value is object-like.
- Do not default malformed required data into a valid value. Defaults are for documented absence, not invalid input.
- Make coercion explicit. String-to-number, date, boolean, and enum coercions can silently broaden a contract.
- Decide whether unknown object keys are rejected, stripped, or preserved according to the protocol. Test that decision.
- Keep normalization in one place. Once decoded, internal callers should not repeatedly check the same nullish or transport cases.
- Treat environment variables as untrusted strings and validate the complete configuration at startup.
- Catch values are `unknown`. Narrow them before reading fields or translate them through a boundary-specific helper.

Use brands sparingly. They are valuable for IDs, currency units, or normalized values with collision risk; they are noise for primitives that cannot realistically be confused. A brand is not runtime validation.
