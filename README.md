# refactory-shadows-ts

TypeScript shadow libraries for the Refactory pipeline. npm packages that expose API-identical Rust-backed wrappers via napi-rs, enabling TypeScript developers to write standard code while testing against real Rust implementations.

**Added in Refactory Supplement v0.3** — extends the shadow library model from Python (PyO3) to TypeScript (napi-rs).

## Shadow Libraries

| Package | TS API | Rust Crate | Product(s) |
|---------|--------|------------|-----------|
| `@refactory/shadow-string-ts` | String methods | Rust `String`/`str` | Sinter (TS SDK) |
| `@refactory/shadow-json-ts` | `JSON.parse/stringify` | `serde_json` | Sinter (TS SDK) |
| `@refactory/shadow-http-ts` | fetch-like API | `reqwest` | Sinter connectors |

## Architecture

TypeScript shadow libraries use napi-rs (Node.js native addon in Rust) to expose Rust-backed implementations with TypeScript-native APIs. The import resolution uses TypeScript path mapping in `tsconfig.json` rather than a runtime import hook.

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

## License

Apache-2.0
