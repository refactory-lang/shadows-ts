# Requirements Checklist: TypeScript Shadow Libraries

**Purpose**: Track completion of all functional requirements and success criteria for the three shadow library packages.
**Created**: 2026-03-13
**Feature**: [spec.md](../spec.md)

## Package Structure & Build

- [ ] CHK001 `rust/@refactory/shadow-string-ts/` directory exists with `Cargo.toml` and napi-rs configuration
- [ ] CHK002 `rust/@refactory/shadow-json-ts/` directory exists with `Cargo.toml` and napi-rs configuration
- [ ] CHK003 `rust/@refactory/shadow-http-ts/` directory exists with `Cargo.toml` and napi-rs configuration
- [ ] CHK004 Each Rust package has a `Cargo.toml` with `crate-type = ["cdylib"]` and `napi` dependency
- [ ] CHK005 Each package has a `package.json` with correct `@refactory` scope, `main`, `types`, and `napi` fields
- [ ] CHK006 `napi build` compiles each package and produces a `.node` native addon
- [ ] CHK007 Build succeeds on macOS aarch64 (Apple Silicon)
- [ ] CHK008 Build succeeds on macOS x86_64 (Intel)
- [ ] CHK009 Build succeeds on Linux x86_64

## shadow-string-ts (FR-001)

- [ ] CHK010 `toUpperCase` and `toLowerCase` implemented and match JS behavior
- [ ] CHK011 `trim`, `trimStart`, `trimEnd` implemented and match JS behavior
- [ ] CHK012 `slice` and `substring` implemented with correct negative index handling
- [ ] CHK013 `includes`, `startsWith`, `endsWith` implemented with optional position argument
- [ ] CHK014 `indexOf` and `lastIndexOf` implemented with optional start position
- [ ] CHK015 `replace` and `replaceAll` implemented for string pattern arguments
- [ ] CHK016 `split` implemented with separator and limit arguments
- [ ] CHK017 `repeat`, `padStart`, `padEnd` implemented with correct length handling
- [ ] CHK018 `charAt`, `charCodeAt`, `concat` implemented
- [ ] CHK019 `match` and `search` implemented for basic string pattern matching
- [ ] CHK020 Unicode strings (emojis, CJK, combining characters) handled correctly
- [ ] CHK021 TypeScript `.d.ts` declarations match `String.prototype` signatures

## shadow-json-ts (FR-002)

- [ ] CHK022 `parse(text)` correctly deserializes objects, arrays, strings, numbers, booleans, null
- [ ] CHK023 `parse(text, reviver)` supports reviver function with correct (key, value) arguments
- [ ] CHK024 `stringify(value)` correctly serializes all JSON-compatible types
- [ ] CHK025 `stringify(value, replacer, space)` supports replacer function, replacer array, and space formatting
- [ ] CHK026 `parse` throws on invalid JSON with informative error messages including position
- [ ] CHK027 Nested objects and arrays (3+ levels deep) serialize and deserialize correctly
- [ ] CHK028 Special values: `undefined` properties omitted, `NaN`/`Infinity` become `null`, `Date` uses `toISOString()`
- [ ] CHK029 TypeScript `.d.ts` declarations match `JSON.parse` and `JSON.stringify` signatures

## shadow-http-ts (FR-003, FR-010)

- [ ] CHK030 `fetch(url)` performs GET request and returns Response with `status`, `ok`, `headers`
- [ ] CHK031 `fetch(url, { method: "POST", body })` sends request body correctly
- [ ] CHK032 PUT, DELETE, PATCH, HEAD, OPTIONS methods supported
- [ ] CHK033 Request headers set via `init.headers` are sent correctly
- [ ] CHK034 `response.json()` returns parsed JSON body as a Promise
- [ ] CHK035 `response.text()` returns raw text body as a Promise
- [ ] CHK036 HTTP redirects (301, 302, 307, 308) followed by default
- [ ] CHK037 Network errors (connection refused, DNS failure) reject the Promise with appropriate error
- [ ] CHK038 Non-2xx responses set `response.ok = false` without rejecting the Promise
- [ ] CHK039 TypeScript `.d.ts` declarations match Web Fetch API signatures (`RequestInit`, `Response`)

## TypeScript Integration (FR-004, FR-007)

- [ ] CHK040 `tsconfig.json` path mappings redirect imports to shadow packages without code changes
- [ ] CHK041 IDE autocompletion works correctly with shadow library type declarations
- [ ] CHK042 TypeScript compiler (`tsc`) reports no type errors when using shadow libraries via path mapping
- [ ] CHK043 Removing path mappings reverts to standard Node.js built-in behavior with no errors

## Error Handling (FR-009)

- [ ] CHK044 Rust panics are caught and converted to JavaScript Error objects
- [ ] CHK045 Error messages from shadow libraries are descriptive and include context
- [ ] CHK046 Missing native addon (`.node` file not found) produces a clear error message with resolution steps

## Equivalence Tests (FR-008)

- [ ] CHK047 `tests/string.equivalence.test.ts` covers at least 20 string operations
- [ ] CHK048 `tests/json.equivalence.test.ts` covers parse, stringify, reviver, replacer, and space formatting
- [ ] CHK049 `tests/http.equivalence.test.ts` covers GET, POST, error handling, headers, and redirects
- [ ] CHK050 Each test invokes both the shadow implementation and the Node.js built-in with the same input
- [ ] CHK051 Each test asserts output equality (deep equality for objects, strict equality for primitives)
- [ ] CHK052 Test suite runs via `npm test` from repo root
- [ ] CHK053 All equivalence tests pass on CI (macOS and Linux)

## Success Criteria Verification

- [ ] CHK054 SC-001: All three packages compile via napi-rs on macOS and Linux
- [ ] CHK055 SC-002: 100% equivalence test pass rate for implemented operations
- [ ] CHK056 SC-003: Existing TS code works with shadow libs via path mapping with zero code changes
- [ ] CHK057 SC-004: Full build completes in under 120 seconds
- [ ] CHK058 SC-005: Packages are publishable to npm with correct metadata
- [ ] CHK059 SC-006: Test coverage meets minimum thresholds (20 string, 4 JSON, 5 HTTP operations)

## Notes

- Check items off as completed: `[x]`
- Add comments or findings inline
- Link to relevant resources or documentation
- Items are numbered sequentially for easy reference
