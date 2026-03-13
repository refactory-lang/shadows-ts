# Requirements Checklist: TypeScript Shadow Libraries

**Feature Branch**: `001-ts-shadow-libs`
**Last Updated**: 2026-03-13

## R1: napi-rs package per shadow module

- [ ] `@refactory/shadow-string-ts` compiles Rust crate via napi-rs into `.node` addon
- [ ] `@refactory/shadow-string-ts` exports `ShadowString` class with full API surface
- [ ] `@refactory/shadow-string-ts` covers: `toUpperCase`, `toLowerCase`, `trim`, `trimStart`, `trimEnd`, `slice`, `substring`, `indexOf`, `lastIndexOf`, `includes`, `startsWith`, `endsWith`, `replace`, `replaceAll`, `split`, `repeat`, `padStart`, `padEnd`, `charAt`, `charCodeAt`, `concat`, `at`, `match`, `search`, `normalize`
- [ ] `@refactory/shadow-string-ts` implements `toString()` and `valueOf()` for native string interop
- [ ] `@refactory/shadow-string-ts` includes complete `.d.ts` type declarations
- [ ] `@refactory/shadow-json-ts` compiles Rust crate via napi-rs into `.node` addon
- [ ] `@refactory/shadow-json-ts` exports `parse(text, reviver?)` function
- [ ] `@refactory/shadow-json-ts` exports `stringify(value, replacer?, space?)` function
- [ ] `@refactory/shadow-json-ts` exports `ShadowJSONError` extending `SyntaxError`
- [ ] `@refactory/shadow-json-ts` handles `undefined`, `Infinity`, `NaN`, `BigInt`, circular references matching `JSON.stringify` behavior
- [ ] `@refactory/shadow-json-ts` uses `serde_json` as its Rust backend
- [ ] `@refactory/shadow-json-ts` includes complete `.d.ts` type declarations
- [ ] `@refactory/shadow-http-ts` compiles Rust crate via napi-rs into `.node` addon
- [ ] `@refactory/shadow-http-ts` exports `fetch(url, init?)` function returning `Promise<Response>`
- [ ] `@refactory/shadow-http-ts` `RequestInit` supports: `method`, `headers`, `body`, `signal`, `redirect`, `keepalive`
- [ ] `@refactory/shadow-http-ts` `Response` exposes: `status`, `statusText`, `ok`, `headers`, `url`, `redirected`, `type`
- [ ] `@refactory/shadow-http-ts` `Response` methods: `json()`, `text()`, `arrayBuffer()`, `blob()`, `clone()`
- [ ] `@refactory/shadow-http-ts` exports `Headers` class with `get`, `set`, `has`, `delete`, `forEach`, `entries`, `keys`, `values`
- [ ] `@refactory/shadow-http-ts` supports `AbortSignal` for request cancellation
- [ ] `@refactory/shadow-http-ts` uses `reqwest` and `tokio` as its Rust backends
- [ ] `@refactory/shadow-http-ts` includes complete `.d.ts` type declarations
- [ ] All three packages support Node.js 18+
- [ ] All three packages build on Linux (x64, arm64), macOS (x64, arm64), and Windows (x64)
- [ ] All three packages use `@napi-rs/cli` for build tooling

## R2: tsconfig path mapping

- [ ] Workspace `tsconfig.json` defines path aliases mapping to shadow packages
- [ ] Path mappings work with `tsc` compilation
- [ ] Path mappings work with esbuild
- [ ] Path mappings work with webpack
- [ ] Path mappings work with vite
- [ ] Example `tsconfig.json` snippet is documented for consumer projects

## R3: Equivalence test suite

- [ ] `packages/shadow-string-ts/__tests__/equivalence.test.ts` exists and covers Scenario 1 operations
- [ ] `packages/shadow-json-ts/__tests__/equivalence.test.ts` exists and covers Scenario 2 operations
- [ ] `packages/shadow-http-ts/__tests__/equivalence.test.ts` exists and covers Scenario 3 operations
- [ ] Each test file runs identical operations through native JS API and shadow implementation
- [ ] Each test file asserts identical return values using deep equality
- [ ] Each test file asserts identical error behavior (type and message pattern)
- [ ] Tests use vitest with `describe`/`it` blocks
- [ ] String tests cover ASCII, unicode (emoji, CJK, combining characters), and empty strings
- [ ] JSON tests cover round-trip of all JSON-legal types and edge cases
- [ ] HTTP tests use a local test server for request/response verification
- [ ] All tests pass via `npm test` from the workspace root

## R4: Build and packaging

- [ ] `npm run build` at workspace root compiles all three napi-rs crates
- [ ] Each package `package.json` has correct `main` field
- [ ] Each package `package.json` has correct `types` field
- [ ] Each package `package.json` has correct `napi` configuration
- [ ] Each package `package.json` has correct `files` field
- [ ] Workspace uses npm workspaces (or pnpm workspaces) for monorepo management
- [ ] CI pipeline runs `npm run build`
- [ ] CI pipeline runs `npm test`
- [ ] CI pipeline runs `cargo test --workspace`

## R5: API surface coverage

- [ ] shadow-string-ts covers all methods listed in R5 of the spec
- [ ] shadow-json-ts covers all functions, options, and error types listed in R5 of the spec
- [ ] shadow-http-ts covers all request/response APIs listed in R5 of the spec

## Success Criteria

- [ ] `npm run build` completes without errors producing `.node` addons for all three packages
- [ ] `tsc --noEmit` reports zero type errors with workspace tsconfig
- [ ] `npm test` exits 0 with all equivalence test suites green
- [ ] `cargo test --workspace` passes all Rust-side unit tests
- [ ] Sample project with tsconfig path aliases compiles and runs correctly
- [ ] Existing TS code works without modification when path mappings are configured
- [ ] Native addons build and tests pass on macOS arm64, Linux x64, and Windows x64
