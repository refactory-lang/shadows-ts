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


### Speckit Workflow

This repo uses [speckit](https://github.com/speckit) for specification-driven development.

- **Specs**: `specs/<NNN-feature-name>/spec.md` — feature specifications
- **Plans**: `specs/<NNN-feature-name>/plan.md` — implementation plans with tasks
- **Checklists**: `specs/<NNN-feature-name>/checklists/` — quality gates
- **Templates**: `.specify/templates/` — spec, plan, task, checklist templates
- **Extensions**: `.specify/extensions/` — verify, sync, review, workflow hooks

**Branch convention**: Feature branches are named `<NNN>-<short-name>` matching the spec directory (e.g., `001-milestone1-pipeline`).

**Issue → Spec flow**: Issues labeled `spec-ready` trigger the `spec-ready-notify` workflow, which assigns Copilot to run the speckit workflow and produce a spec + plan + tasks.
