# falsegreen-js (JavaScript / TypeScript)

[![CI](https://github.com/vinicq/falsegreen-js/actions/workflows/ci.yml/badge.svg)](https://github.com/vinicq/falsegreen-js/actions/workflows/ci.yml)
[![npm](https://img.shields.io/npm/v/falsegreen-js.svg)](https://www.npmjs.com/package/falsegreen-js)
[![Downloads](https://img.shields.io/npm/dm/falsegreen-js.svg)](https://www.npmjs.com/package/falsegreen-js)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/vinicq/falsegreen-js/blob/main/LICENSE)

The deterministic JS/TS scanner. A static scan via the TypeScript compiler API, no code
execution, runner-agnostic across Jest, Vitest, Mocha+Chai, Jasmine, AVA, node:test, Cypress,
Playwright, and Testing Library. Covers `.js`, `.jsx`, `.ts`, `.tsx`, `.mjs`, `.cjs`, `.mts`,
`.cts`.

- Repository: [github.com/vinicq/falsegreen-js](https://github.com/vinicq/falsegreen-js)
- Catalog: [JavaScript / TypeScript codes](../catalog/javascript-typescript.md)

## Install

```bash
npm install --save-dev falsegreen-js
```

## Use

```bash
npx falsegreen-js src                       # scan
npx falsegreen-js --format json|sarif|junit # output shape (matches the Python sibling)
npx falsegreen-js --baseline .falsegreen-baseline.json
npx falsegreen-js --config-audit            # jest/vitest config for project-level false-green
```

The JSON report carries `riskGroup` and `oracleRegistryVersion`: codes are classified into closed
risk groups, and the oracle registry (sync-fail / promise / runner-registered / value-only) is
versioned so async detection is principled rather than string-matched.

## First finding

Save a test whose assertion never runs:

```ts
// demo.test.ts
import { add } from "./add";

it("adds", () => {
  expect(add(2, 2));   // no matcher - this asserts nothing
});
```

Run the scanner over it:

```bash
npx falsegreen-js demo.test.ts
```

It reports:

```
demo.test.ts:5  [JS2] expect() with no matcher - the assertion never runs
    level: unit   fix: add a matcher, e.g. .toBe(4)

Summary: 1 high, 0 low.
```

## Reading a finding

Each line carries the same fields:

- `demo.test.ts:5` - the file and the line that triggered it.
- `[JS2]` - the catalog code. `JS2` is `expect()` with no matcher. Every code is
  explained in the [JS/TS catalog](../catalog/javascript-typescript.md).
- `level: unit` - which level of the [test pyramid](../concepts/pyramid.md) the
  file sits at, read from the file's imports.
- `fix:` - a one-line hint. Here: add a matcher so the assertion actually runs.

`--format json|sarif|junit` gives a machine-readable report; SARIF uploads to
GitHub code scanning so findings show inline on the pull request.

## What it covers

Full per-code detail in the [JS/TS catalog](../catalog/javascript-typescript.md).

| Group | Codes | Effect |
|---|---|---|
| **Shared with Python** (F1-F6) | `C2`, `C2b`, `C5`, `C6`, `C7`, `C8`, `C9`, `C16`, `C18`, `C20`, `C21`, `C23`, `C37`, `C44`, `CC` | HIGH blocks, LOW warns |
| **JavaScript-specific** | `JS1`-`JS9`, `JS11`, `JS13`, `JS15`, `JS17`, `JS18`, `JS21`, `JS22` | idem |
| **Diagnostic / coupling** (F8) | `D1`, `D3`, `D4`, `D6`, `D7`, `D8`, `M2` | opt-in |
| **Project / CI** (`--config-audit`) | `PL10` (`--passWithNoTests`), `PL7` (coverage threshold), `PL8` (bail) | reads jest/vitest config |

## What it does not cover, and why

### Out of scope (the wrong axis)
The same boundary as the rest of the family: brittleness/false-red, hygiene, slow, design,
naming, duplication, runtime. See [coverage vs the literature](../concepts/denominator.md).

### JS codes deliberately not implemented
| Code | What it would flag | Why not |
|---|---|---|
| **JS10** | any conditional in the test body | too broad; `jest/no-conditional-in-test` (ESLint) covers it. Diagnostic-only at most |
| **JS12** | a promise with `expect` not returned/awaited | subsumed by `JS7` |
| **JS14** | a giant snapshot | hygiene (F8), not false-green |
| **JS16** | async test with no `expect.assertions(n)` guard | the absence of a guard is not a smell; flagging it has a very high false-positive rate |
| **JS19** | `toBe` on an object/array literal (was `toEqual`) | this is false-RED (an identity check that fails on correct code): the opposite axis |
| **JS20** | a Promise compared without `resolves`/`rejects` | knowing the subject is a Promise needs type information; high false positive |

### Not applicable from Python
pytest-specific codes have no JS analog: the collection rules (`C4` family), `pytest.raises`
binding/scope codes, `capsys`/`os.environ` codes. They are intentionally absent, not missing.

### Beyond the scanner
The semantic slice (intent, oracle correctness) is [falsegreen-skill](skill.md); runtime
(mutation with Stryker) is out of band. See [scope and honesty](../concepts/scope.md).
