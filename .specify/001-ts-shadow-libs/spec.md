# Feature Specification: TypeScript Shadow Libraries

**Feature Branch**: `001-ts-shadow-libs`
**Created**: 2026-03-13
**Status**: Draft

## Overview

Three TypeScript shadow libraries provide API-identical wrappers over Rust implementations compiled to native Node.js addons via napi-rs. Standard TypeScript code uses familiar APIs (`String` methods, `JSON.parse`/`JSON.stringify`, `fetch`), while `tsconfig.json` path mappings transparently redirect imports to the Rust-backed packages. No application code changes are required.

### Shadow Packages

| Package                       | TS/JS API it shadows       | Rust backend     |
|-------------------------------|----------------------------|------------------|
| `@refactory/shadow-string-ts` | `String.prototype` methods | `String` / `str` |
| `@refactory/shadow-json-ts`   | `JSON.parse`, `JSON.stringify` | `serde_json` |
| `@refactory/shadow-http-ts`   | `fetch`-like API           | `reqwest`        |

### Repository Layout

```
shadows-ts/
  packages/
    shadow-string-ts/
      Cargo.toml              # napi-rs, napi-derive
      src/lib.rs              # #[napi] functions for string operations
      index.ts                # TS re-exports wrapping napi bindings
      index.d.ts              # Type declarations matching String.prototype
      package.json
      __tests__/
        equivalence.test.ts
    shadow-json-ts/
      Cargo.toml              # napi-rs, serde_json
      src/lib.rs
      index.ts
      index.d.ts
      package.json
      __tests__/
        equivalence.test.ts
    shadow-http-ts/
      Cargo.toml              # napi-rs, reqwest, tokio
      src/lib.rs
      index.ts
      index.d.ts
      package.json
      __tests__/
        equivalence.test.ts
  tsconfig.json               # Path mappings for import redirection
  tsconfig.base.json          # Shared compiler options
  package.json                # Workspace root
  vitest.config.ts            # Test runner configuration
```

---

## User Scenarios & Testing

### Scenario 1: String operations via shadow

A developer writes standard string manipulation code:

```typescript
import { ShadowString } from "@refactory/shadow-string-ts";

const s = new ShadowString("Hello, World!");
console.log(s.toUpperCase());       // "HELLO, WORLD!"
console.log(s.slice(0, 5));          // "Hello"
console.log(s.includes("World"));    // true
console.log(s.replace("World", "Rust")); // "Hello, Rust!"
console.log(s.split(", "));         // ["Hello", "World!"]
```

With `tsconfig.json` path mapping, a project can alias `string-utils` or similar to `@refactory/shadow-string-ts` so that existing utility imports resolve to the shadow.

**Test**: `packages/shadow-string-ts/__tests__/equivalence.test.ts`
- For each method, run the same operation on a native JS `String` and a `ShadowString`.
- Compare return values for: `toUpperCase`, `toLowerCase`, `trim`, `trimStart`, `trimEnd`, `slice`, `substring`, `indexOf`, `lastIndexOf`, `includes`, `startsWith`, `endsWith`, `replace`, `replaceAll`, `split`, `repeat`, `padStart`, `padEnd`, `charAt`, `charCodeAt`, `concat`, `match`, `search`.
- Test with ASCII, unicode (emoji, CJK, combining characters), and empty strings.

### Scenario 2: JSON parsing and serialization

```typescript
import { parse, stringify } from "@refactory/shadow-json-ts";

const data = { key: [1, 2.5, true, null], nested: { a: "b" } };
const encoded = stringify(data, null, 2);
const decoded = parse(encoded);
// decoded deep-equals data
```

The `tsconfig.json` maps a project-local `json-utils` path (or the developer imports `@refactory/shadow-json-ts` directly).

**Test**: `packages/shadow-json-ts/__tests__/equivalence.test.ts`
- Round-trip primitives: strings, numbers (integer, float, negative, zero), booleans, null.
- Round-trip complex structures: nested objects, arrays, mixed types.
- `stringify` options: replacer (function and array), space (number and string).
- `parse` options: reviver function.
- Error cases: malformed JSON throws `SyntaxError`-compatible error with similar message.
- Edge cases: `Infinity`, `NaN`, `undefined` handling matches `JSON.stringify` behavior (outputs `null` or omits keys).

### Scenario 3: HTTP fetch

```typescript
import { fetch } from "@refactory/shadow-http-ts";

const response = await fetch("https://api.example.com/data", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ query: "test" }),
});

const json = await response.json();
console.log(response.status); // 200
```

**Test**: `packages/shadow-http-ts/__tests__/equivalence.test.ts`
- Use a local HTTP test server (via `msw` or a simple `http.createServer`).
- GET, POST, PUT, DELETE, PATCH requests.
- Request headers, query parameters, JSON body, text body.
- Response: `.status`, `.statusText`, `.headers`, `.json()`, `.text()`, `.arrayBuffer()`.
- Timeout handling, abort signal support.
- Compare behavior against Node.js built-in `fetch` (Node 18+).

### Scenario 4: tsconfig path mapping

A project's `tsconfig.json` includes:

```json
{
  "compilerOptions": {
    "paths": {
      "@app/string-utils": ["./node_modules/@refactory/shadow-string-ts"],
      "@app/json": ["./node_modules/@refactory/shadow-json-ts"],
      "@app/http": ["./node_modules/@refactory/shadow-http-ts"]
    }
  }
}
```

Existing imports like `import { parse } from "@app/json"` now resolve to the shadow implementation without changing source files.

**Test**: Create a small integration test project with the path mappings above. Verify that `tsc --noEmit` type-checks successfully and that runtime behavior matches native APIs.

---

## Requirements

### R1: napi-rs package per shadow module

Each of the three packages must:

1. Compile a Rust crate via napi-rs into a platform-specific `.node` native addon.
2. Export a TypeScript API that mirrors the standard JS/TS API it shadows.
3. Include full `.d.ts` type declarations so that consumers get accurate IntelliSense and type checking.
4. Support Node.js 18+ on Linux (x64, arm64), macOS (x64, arm64), and Windows (x64).
5. Use `@napi-rs/cli` for build tooling.

### R2: tsconfig path mapping

1. The workspace `tsconfig.json` must define path aliases that map standard-looking imports to shadow packages.
2. Path mappings must work with both `tsc` compilation and runtime bundlers (esbuild, webpack, vite).
3. A documented example `tsconfig.json` snippet must be provided showing how consumers configure their own projects.

### R3: Equivalence test suite

Each package must include `__tests__/equivalence.test.ts` that:

1. Runs the same operations through the native JS/TS API and the shadow implementation.
2. Asserts identical return values using deep equality.
3. Asserts identical error behavior (error type, message pattern).
4. Uses `vitest` as the test runner with `describe`/`it` blocks.
5. Is runnable via `npm test` from the workspace root.

### R4: Build and packaging

1. `npm run build` at the workspace root compiles all three napi-rs crates and produces the `.node` addons.
2. Each package has its own `package.json` with correct `main`, `types`, `napi`, and `files` fields.
3. The workspace uses npm workspaces (or pnpm workspaces) for monorepo management.
4. CI must run: `npm run build`, `npm test`, and `cargo test --workspace`.

### R5: API surface — minimum coverage per package

**@refactory/shadow-string-ts**: `toUpperCase`, `toLowerCase`, `trim`, `trimStart`, `trimEnd`, `slice`, `substring`, `indexOf`, `lastIndexOf`, `includes`, `startsWith`, `endsWith`, `replace`, `replaceAll`, `split`, `repeat`, `padStart`, `padEnd`, `charAt`, `charCodeAt`, `concat`, `at`, `match`, `search`, `normalize`. Constructor accepts `string`. Implements `toString()` and `valueOf()` for interop with native strings.

**@refactory/shadow-json-ts**: `parse(text: string, reviver?: Function): any`, `stringify(value: any, replacer?: Function | string[], space?: number | string): string`. Error class `ShadowJSONError` extending `SyntaxError` for parse failures. Handles all JSON-legal types; behavior on `undefined`, `Infinity`, `NaN`, `BigInt`, and circular references must match `JSON.stringify`.

**@refactory/shadow-http-ts**: `fetch(url: string | URL, init?: RequestInit): Promise<Response>`. `RequestInit` supports `method`, `headers`, `body` (string, Buffer, ReadableStream), `signal` (AbortSignal), `redirect`, `keepalive`. `Response` exposes `status`, `statusText`, `ok`, `headers`, `url`, `redirected`, `type`, and methods `json()`, `text()`, `arrayBuffer()`, `blob()`, `clone()`. `Headers` class with `get`, `set`, `has`, `delete`, `forEach`, `entries`, `keys`, `values`.

---

## Success Criteria

1. **Build succeeds**: `npm run build` at the workspace root completes without errors, producing `.node` addons for all three packages.
2. **Type checking passes**: `tsc --noEmit` with the workspace `tsconfig.json` reports zero errors.
3. **Equivalence tests pass**: `npm test` exits 0 with all `equivalence.test.ts` suites green across all three packages.
4. **Cargo tests pass**: `cargo test --workspace` in the Rust workspace passes all native unit tests.
5. **Path mapping works**: A sample project with `tsconfig.json` path aliases compiles and runs correctly, loading shadow implementations via the mapped paths.
6. **No source changes required**: Existing TS code using `String` methods, `JSON.parse`/`JSON.stringify`, or `fetch` works without modification when path mappings are configured.
7. **Cross-platform**: Native addons build and tests pass on macOS arm64 (development), Linux x64 (CI), and Windows x64 (CI).
