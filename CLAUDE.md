<!-- codemod-skill-discovery:begin -->
## Codemod Skill Discovery
This section is managed by `codemod` CLI.

- Core skill: `.agents/skills/codemod/SKILL.md`
- Package skills: `.agents/skills/<package-skill>/SKILL.md`
- List installed Codemod skills: `npx codemod agent list --harness antigravity --format json`

<!-- codemod-skill-discovery:end -->

## Project: shadows-ts

TypeScript shadow libraries for the Refactory pipeline. Part of the [refactory-lang](https://github.com/refactory-lang) organization. Provides API-identical TypeScript/Node.js wrappers backed by Rust implementations via napi-rs native addons. Uses TypeScript path mapping (not a runtime hook) for import resolution.

### Architecture

- **Packages** (`packages/`): TypeScript shadow library packages
  - `string-ts/`: String methods -> Rust `String`/`str`
  - `json-ts/`: `JSON.parse/stringify` -> `serde_json`
  - `http-ts/`: fetch-like API -> `reqwest`
- **Rust** (`rust/`): napi-rs native addon implementations
- **Tests** (`tests/`): Shared equivalence tests (native TS vs. shadow)
- **Specs** (`specs/`): Implementation specifications

### Running

```bash
# Build native addons
cd rust && cargo build

# Run equivalence tests
npm test
```

### Key Files

| File | Purpose |
|------|---------|
| `packages/string-ts/` | String method shadow (`@refactory/shadow-string-ts`) |
| `packages/json-ts/` | JSON shadow (`@refactory/shadow-json-ts`) |
| `packages/http-ts/` | HTTP/fetch shadow (`@refactory/shadow-http-ts`) |
| `rust/` | napi-rs native addon Rust source |
| `tests/` | Equivalence test suite |

### Conventions

- Import resolution uses `tsconfig.json` path mapping, not a runtime hook (unlike shadows-python)
- Each shadow package is scoped under `@refactory/`
