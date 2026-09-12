# Compiler and runtime

Read this reference when creating or editing `tsconfig.json`, changing runtimes, or diagnosing module/typecheck behavior.

## Choose settings from the execution model

- Bundled application: follow the framework and bundler; commonly use `moduleResolution: "bundler"`, ESM-preserving output, and `noEmit: true` when another tool emits.
- JavaScript emitted and run directly by Node: use `module: "nodenext"` and make the package module type explicit.
- Published library: use the strictest configuration compatible with the lowest promised runtime; see [package output](package-output.md).
- Native Node TypeScript: treat it as execution by type stripping, not type checking. Run `tsc --noEmit` separately and use only erasable syntax unless a supported transformer is intentionally present.

Do not copy a `tsconfig` between these modes without re-evaluating it. Split browser, server, worker, test, and shared-code projects when they require different globals or resolution behavior.

## Strict baseline

Respect an existing repository configuration. For a new project or an intentional strictness upgrade, start from the current compiler defaults and consider this baseline:

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedSideEffectImports": true,
    "noImplicitOverride": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "verbatimModuleSyntax": true,
    "types": []
  }
}
```

Add only the ambient type packages each project needs. Under TypeScript 6.0, `types` defaults to `[]`; list values such as `node` only where their globals are valid. Set `rootDir` explicitly when emitting from a nested source directory.

`exactOptionalPropertyTypes` preserves the runtime distinction between a missing key and a key present with `undefined`. `noUncheckedIndexedAccess` makes unchecked keys and array indexes honestly nullable. Fix the resulting uncertainty by changing the model, checking bounds/keys, or narrowing—not with `!`.

## TypeScript 6.0 guidance

- Prefer syntax that can be erased without generating JavaScript: literal unions or `as const` objects instead of enums, ECMAScript modules instead of namespaces, and explicit class fields instead of parameter properties. Require `erasableSyntaxOnly` when Node executes `.ts` directly; enable it elsewhere only as an intentional project constraint.
- Do not add deprecated `baseUrl`; write explicit `paths` entries or use package `imports`.
- Do not use `moduleResolution: "node"`/`"node10"`; choose `nodenext` for Node or `bundler` for bundler-controlled output.
- Target the actual supported runtime. Do not emit ES5 with TypeScript 6.0.
- Do not enable `stableTypeOrdering` as a general setting; it is a temporary TypeScript 6-to-7 migration diagnostic and can slow checking.
- Use type-only imports where required under `verbatimModuleSyntax`. Do not rely on import elision to hide a runtime import.
- Avoid namespace-era module patterns in application code. Use ECMAScript modules.

## Linting and formatting

Use the repository's configured tool rather than introducing Biome or ESLint during an unrelated change. When type-aware linting is available, enforce at least unsafe `any` usage, floating or misused promises, non-null assertions, and unsafe assignments/calls/returns. Formatting and linting complement `tsc`; neither replaces it.

Do not enable `skipLibCheck` merely to conceal errors in project-owned declarations. In applications it can be a deliberate performance tradeoff for third-party declaration conflicts; libraries should test their emitted declarations under supported consumer configurations.
