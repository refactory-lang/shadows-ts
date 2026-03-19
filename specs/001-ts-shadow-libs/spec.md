# Feature Specification: TypeScript Shadow Libraries

**Feature Branch**: `001-ts-shadow-libs`
**Created**: 2026-03-13
**Status**: Draft
**Input**: User description: "Implement TypeScript shadow libraries - shadow-string-ts (Rust String), shadow-json-ts (serde_json), shadow-http-ts (reqwest) with napi-rs bindings and Node.js equivalence tests"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - String Operations via Rust-Backed Shadow Library (Priority: P1)

A developer writes standard TypeScript string code using familiar methods like `slice`, `toUpperCase`, `includes`, `replace`, `split`, and `trim`. Their `tsconfig.json` has path mappings that redirect string imports to `@refactory/shadow-string-ts`. At build time and runtime, these calls are routed to a napi-rs native addon backed by Rust's `String` and `str` types. The developer's code remains unchanged -- they write normal TypeScript and get Rust performance characteristics transparently.

**Why this priority**: String manipulation is the most common primitive operation in any application. It exercises the full napi-rs binding pipeline (TS type declarations, native addon compilation, runtime invocation) with simple, synchronous semantics, making it the ideal first proof-of-concept for the shadow library architecture.

**Independent Test**: Can be fully tested by importing string shadow functions, running standard string operations, and asserting output matches Node.js built-in string behavior. Delivers a working napi-rs build pipeline and the foundational pattern for all other shadow libraries.

**Acceptance Scenarios**:

1. **Given** a TypeScript file that calls `shadowString.toUpperCase("hello")`, **When** the project is compiled and run with tsconfig path mapping pointing to `@refactory/shadow-string-ts`, **Then** the result is `"HELLO"`, identical to native JS `"hello".toUpperCase()`.
2. **Given** a string containing Unicode characters (e.g., emojis, CJK), **When** `shadowString.slice(str, 0, 3)` is called, **Then** the result matches the behavior of JS `str.slice(0, 3)` for the same input.
3. **Given** the `@refactory/shadow-string-ts` npm package, **When** a developer runs `npm run build` in the `rust/` directory, **Then** the napi-rs build completes successfully and produces a `.node` native addon file.
4. **Given** a TypeScript project with no shadow library imports, **When** the developer adds tsconfig path mappings for `@refactory/shadow-string-ts`, **Then** their existing string code continues to compile and produce identical results.

---

### User Story 2 - JSON Parse/Stringify via serde_json (Priority: P2)

A developer uses `JSON.parse()` and `JSON.stringify()` in their TypeScript code. Through tsconfig path mapping, these calls are redirected to `@refactory/shadow-json-ts`, which delegates to Rust's `serde_json` crate via napi-rs. The shadow library handles serialization and deserialization of objects, arrays, nested structures, and primitive values, returning results identical to the built-in `JSON` global.

**Why this priority**: JSON parsing is the second most common operation after string manipulation, especially in web services and API layers. It validates that the shadow library architecture can handle complex, nested data structures crossing the JS-Rust boundary via napi-rs.

**Independent Test**: Can be tested by parsing JSON strings and stringifying objects through the shadow library, then comparing output byte-for-byte against Node.js built-in `JSON.parse` and `JSON.stringify`.

**Acceptance Scenarios**:

1. **Given** a valid JSON string `'{"name":"alice","age":30}'`, **When** `shadowJson.parse(input)` is called, **Then** the returned object is deeply equal to `JSON.parse('{"name":"alice","age":30}')`.
2. **Given** a JavaScript object with nested arrays and objects, **When** `shadowJson.stringify(obj)` is called, **Then** the output string is identical to `JSON.stringify(obj)`.
3. **Given** an invalid JSON string, **When** `shadowJson.parse(badInput)` is called, **Then** the function throws an error with a message that includes the position of the syntax error.
4. **Given** a `stringify` call with a `replacer` function and `space` argument, **When** `shadowJson.stringify(obj, replacer, 2)` is called, **Then** the formatted output matches `JSON.stringify(obj, replacer, 2)`.

---

### User Story 3 - HTTP Fetch via reqwest (Priority: P3)

A developer uses a fetch-like API (modeled on the Web Fetch standard) in their TypeScript code. Through tsconfig path mapping (`node-fetch` or `fetch` mapped to `@refactory/shadow-http-ts`), HTTP requests are handled by Rust's `reqwest` crate via napi-rs. The shadow library supports GET, POST, PUT, DELETE methods, request headers, JSON/text response bodies, and status code handling.

**Why this priority**: HTTP client functionality is the most complex shadow library due to async I/O, but it demonstrates that the architecture works for asynchronous operations crossing the JS-Rust boundary. It is lower priority because it depends on the foundational napi-rs patterns proven in P1 and P2.

**Independent Test**: Can be tested by making HTTP requests to a local test server (or mock), comparing response status codes, headers, and body content against equivalent calls made with Node.js native `fetch` or `node-fetch`.

**Acceptance Scenarios**:

1. **Given** a running HTTP server that returns `{"ok":true}` at `/health`, **When** `shadowHttp.fetch("http://localhost:PORT/health")` is called, **Then** the response has status 200 and `response.json()` resolves to `{ok: true}`.
2. **Given** a POST endpoint that echoes the request body, **When** `shadowHttp.fetch(url, { method: "POST", body: JSON.stringify({key: "value"}), headers: {"Content-Type": "application/json"} })` is called, **Then** the echoed body matches the sent body.
3. **Given** a URL that returns a 404, **When** `shadowHttp.fetch(url)` is called, **Then** `response.ok` is `false` and `response.status` is `404`.
4. **Given** a URL that is unreachable (connection refused), **When** `shadowHttp.fetch(url)` is called, **Then** the promise rejects with a network error.

---

### User Story 4 - Equivalence Test Suite (Priority: P1)

A CI pipeline runs a shared equivalence test suite that exercises every shadow library function side-by-side with its Node.js built-in counterpart. Each test calls both the shadow implementation and the native implementation with identical inputs and asserts that outputs match. The test suite lives in `tests/` and is parameterized to run against any target backend (currently Rust).

**Why this priority**: Without equivalence tests, there is no way to verify that the shadow libraries are truly API-identical to their Node.js counterparts. This is a P1 because it is the acceptance gate for all other stories.

**Independent Test**: Can be run independently with `npm test` from the repo root. Each test file covers one shadow library and produces pass/fail results showing exactly which operations diverge, if any.

**Acceptance Scenarios**:

1. **Given** the equivalence test suite and a built `@refactory/shadow-string-ts` package, **When** `npm test` is run, **Then** all string equivalence tests pass, confirming identical behavior to native JS string methods.
2. **Given** the equivalence test suite and a built `@refactory/shadow-json-ts` package, **When** `npm test` is run, **Then** all JSON equivalence tests pass, confirming identical behavior to native `JSON.parse` and `JSON.stringify`.
3. **Given** a new string method is added to the shadow library, **When** the developer does not add a corresponding equivalence test, **Then** a coverage check or linting rule flags the missing test.

---

### Edge Cases

- What happens when a Rust string operation encounters invalid UTF-8 sequences that JavaScript handles permissively (e.g., lone surrogates)? The shadow library must either match JS behavior or document the divergence explicitly.
- How does `JSON.parse` handle `__proto__` keys, `BigInt` values, or circular references? The serde_json-backed implementation must replicate V8's behavior for `__proto__` and throw on unsupported types.
- What happens when `fetch` encounters HTTP redirects (301, 302, 307, 308)? The reqwest-backed implementation must follow redirects by default, matching fetch spec behavior.
- How does the system handle the native addon `.node` file not being found at runtime (e.g., wrong architecture, missing build step)? The shadow library should throw a clear error indicating the native addon is missing and suggesting `npm run build`.
- What happens when tsconfig path mappings are removed? The TypeScript code should fall back to standard Node.js built-in behavior with no runtime errors.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST provide an npm package `@refactory/shadow-string-ts` that exposes functions mirroring JavaScript's `String.prototype` methods (`slice`, `substring`, `toUpperCase`, `toLowerCase`, `trim`, `trimStart`, `trimEnd`, `includes`, `indexOf`, `lastIndexOf`, `startsWith`, `endsWith`, `replace`, `replaceAll`, `split`, `repeat`, `padStart`, `padEnd`, `charAt`, `charCodeAt`, `concat`, `match`, `search`).
- **FR-002**: System MUST provide an npm package `@refactory/shadow-json-ts` that exposes `parse(text: string, reviver?: Function): any` and `stringify(value: any, replacer?: Function | string[], space?: number | string): string`, matching the signatures of the built-in `JSON` global.
- **FR-003**: System MUST provide an npm package `@refactory/shadow-http-ts` that exposes a `fetch(url: string, init?: RequestInit): Promise<Response>` function supporting GET, POST, PUT, DELETE, PATCH, HEAD, and OPTIONS methods, request headers, and JSON/text response body reading.
- **FR-004**: Each shadow library package MUST include TypeScript declaration files (`.d.ts`) that exactly match the type signatures of their Node.js counterparts, enabling full IDE autocompletion and type checking.
- **FR-005**: Each shadow library MUST be implemented as a napi-rs native addon, compiled from Rust source code located under `rust/@refactory/shadow-<name>-ts/`.
- **FR-006**: The build system MUST use `napi-rs` (via `@napi-rs/cli`) to compile Rust code into a platform-specific `.node` native addon, with support for at least macOS (aarch64, x86_64) and Linux (x86_64).
- **FR-007**: Import redirection MUST work via `tsconfig.json` `paths` mapping (e.g., mapping `"string-utils"` to `["@refactory/shadow-string-ts"]`) with no runtime hook or loader required.
- **FR-008**: The `tests/` directory MUST contain equivalence tests that invoke both the shadow library and the Node.js built-in for each operation and assert identical output.
- **FR-009**: Each shadow library MUST throw errors that are compatible with standard JavaScript Error types, including appropriate error messages and stack traces.
- **FR-010**: The `@refactory/shadow-http-ts` package MUST support async/await patterns, returning Promises that resolve or reject in the same circumstances as the Web Fetch API.

### Key Entities

- **Shadow Library Package**: An npm package under the `@refactory` scope containing TypeScript declarations and a compiled napi-rs native addon. Each package maps one-to-one with a standard TS/JS API surface (String methods, JSON global, fetch API).
- **Native Addon**: A `.node` file compiled from Rust via napi-rs. It is the binary artifact that the shadow library's TypeScript wrapper loads at runtime. Platform-specific (OS + architecture).
- **Equivalence Test**: A test case that calls both the shadow implementation and the Node.js built-in with the same input, then asserts output equality. Lives in `tests/` and is parameterized by target backend.
- **tsconfig Path Mapping**: A compiler configuration entry that redirects TypeScript module resolution from a standard import path to a shadow library package, enabling transparent substitution without code changes.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: All three shadow library packages (`shadow-string-ts`, `shadow-json-ts`, `shadow-http-ts`) compile successfully via `napi-rs` on macOS and Linux, producing valid `.node` native addons.
- **SC-002**: The equivalence test suite passes with 100% of implemented operations producing output identical to their Node.js built-in counterparts when given the same input.
- **SC-003**: A developer can take an existing TypeScript file that uses `String.prototype` methods, `JSON.parse/stringify`, or `fetch`, add tsconfig path mappings, and run it against the shadow libraries with zero code changes to the application source.
- **SC-004**: The `npm run build` command in the repo root completes in under 120 seconds on a modern development machine (M1/M2 Mac or equivalent).
- **SC-005**: Each shadow library package is publishable to npm with correct `main`, `types`, and `napi` fields in `package.json`, and can be installed and used in a fresh TypeScript project.
- **SC-006**: The equivalence test suite covers at least 20 string operations, 4 JSON operations (parse, stringify with variations), and 5 HTTP operations (GET, POST, error handling, headers, redirect following).

---

## v0.3 Addendum: Extended Scope

*Added 2026-03-16 to align with master spec v0.3*

### Expanded Library Inventory

The initial 3 libraries (shadow-string-ts, shadow-json-ts, shadow-http-ts) represent the Milestone 1 minimum. The full shadow-ts ecosystem should expand to cover the standard Node.js modules used in n8n nodes and Sinter steps:

| Library | Backing Rust Crate | Milestone |
|---------|-------------------|-----------|
| `shadow-string-ts` | Rust `String`/`str` | Milestone 1 |
| `shadow-json-ts` | `serde_json` | Milestone 1 |
| `shadow-http-ts` | `reqwest` | Milestone 1 |
| `shadow-path-ts` | `std::path` | Milestone 2 |
| `shadow-crypto-ts` | `sha2`/`ring` | Milestone 2 |
| `shadow-url-ts` | `url` | Milestone 2 |
| `shadow-buffer-ts` | `Vec<u8>` | Milestone 2 |
| `shadow-fs-ts` | `std::fs`/`tokio::fs` | Milestone 2 |

### Fallback Strategy

When a TypeScript file imports a module with no shadow:
- **Profile validator warns** on unrecognised imports (Category B conditional — not a hard reject)
- **Normalize-Det flags** the import for manual review
- If a clear Rust crate equivalent exists, it surfaces as a **shadow library stub request** in the promotion pipeline
- If no equivalent exists, the import is a **hard profile violation**

### Additional Success Criteria

- **SC-007**: Fallback strategy produces clear, actionable diagnostics for unshadowed imports (module name, suggested Rust crate, resolution options)
- **SC-008**: Milestone 2 libraries achieve equivalence test coverage of at least 15 operations each
