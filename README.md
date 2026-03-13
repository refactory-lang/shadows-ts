# shadows-ts

TypeScript shadow libraries for the Refactory pipeline. API-identical wrappers for TypeScript/Node.js APIs, backed by target-language implementations via native addons.

## Structure

```
shadows-ts/
├── rust/                              # Target: Rust (napi-rs)
│   ├── @refactory/shadow-string-ts/   # String methods → Rust String/str
│   ├── @refactory/shadow-json-ts/     # JSON.parse/stringify → serde_json
│   └── @refactory/shadow-http-ts/     # fetch-like API → reqwest
├── go/                                # Target: Go (future)
├── tests/                             # Shared equivalence tests
└── README.md
```

## How It Works

TypeScript shadow libraries use napi-rs (Node.js native addon in Rust) or equivalent FFI for other targets. Import resolution uses TypeScript path mapping rather than a runtime hook:

```json
// tsconfig.json
{
  "compilerOptions": {
    "paths": {
      "node-fetch": ["@refactory/shadow-http-ts"]
    }
  }
}
```

## Shadow Libraries (Rust target)

| Package | TS API | Rust Crate |
|---------|--------|------------|
| `@refactory/shadow-string-ts` | String methods | Rust `String`/`str` |
| `@refactory/shadow-json-ts` | `JSON.parse/stringify` | `serde_json` |
| `@refactory/shadow-http-ts` | fetch-like API | `reqwest` |

## Adding a New Target Language

Create a new directory at the root (e.g. `go/`) with the target-specific build system. The equivalence test templates are shared.

## License

Apache-2.0
