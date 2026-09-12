# Package output

Read this reference when publishing a library, CLI, or package that other projects execute or typecheck.

## Define the contract

- State the minimum supported runtime and module formats. Prefer one format when consumers permit it; dual ESM/CommonJS output creates two contracts and must be tested as two contracts.
- Set `package.json` `type` explicitly. Use `exports` to enumerate supported public entry points and keep internal modules private.
- Export project-owned values and types from deliberate entry points. Do not expose internal file layout or third-party implementation types accidentally.
- Generate declarations and declaration maps for typed consumers. Check the emitted `.d.ts` files, not just source inference.
- Target the oldest runtime promised to consumers, not the newest runtime on the author's machine.

## Node and module resolution

For JavaScript emitted and run by Node, use `module: "nodenext"`. In ESM source, write the runtime `.js` extension in relative imports inside `.ts` files:

```ts
export { createClient } from "./client.js";
```

This is intentional: Node ESM does not search for extensions, and the emitted file is `.js`. Do not apply this rule blindly to bundler-only applications; follow their configured resolution mode.

Do not validate a published library only with `moduleResolution: "bundler"`. It can accept specifiers that work in a bundler but fail when the emitted JavaScript runs directly in Node.

Native Node execution of `.ts` files strips erasable types and ignores `tsconfig.json`; it does not typecheck. Publish JavaScript plus declarations by default unless the package contract explicitly requires TypeScript source and every supported consumer can execute it.

## Release proof

Use the repository's package manager and existing tools. As applicable:

1. Run the clean build and declaration emit.
2. Create the actual package tarball and inspect its file list, `exports`, executable permissions, source maps, and declarations.
3. Install the tarball into a temporary consumer and execute each promised entry point and CLI under supported Node versions.
4. Typecheck representative consumers, including ESM and CommonJS only when both are promised.
5. Run Publint and Are The Types Wrong when available or when adding them is approved.
6. Check unused files and exports with the repository's configured tool, such as Knip, without deleting API surface solely because static analysis cannot see a consumer.

Source tests, direct TypeScript execution, and successful declaration emit do not prove that packed runtime output works. Treat the installed tarball as the release artifact.
