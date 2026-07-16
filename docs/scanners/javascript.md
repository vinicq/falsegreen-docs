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

## Complete usage and configuration { #complete-usage-and-configuration }

The getting-started above is the five-minute path. This section is the full reference:
every install channel, every output format, every configuration knob, the exit-code
contract, and the CI wiring. It mirrors what the [project README](https://github.com/vinicq/falsegreen-js)
documents.

### Install channels { #install-channels }

Node 18 or newer. The scanner reads `.js`, `.jsx`, `.ts`, `.tsx`, `.mjs`, `.cjs`, `.mts`,
`.cts`.

```bash
npm install -D falsegreen-js   # project dev dependency
npm install -g falsegreen-js   # global install
npx falsegreen-js .            # run the latest from npm without installing
node dist/cli.js .             # run from a cloned checkout
```

Runner-agnostic: the assertion and test vocabulary spans Jest, Vitest, Mocha + Chai, Jasmine,
AVA, `node:test`, tap, Cypress, Playwright, Testing Library, and Vue Test Utils, so a Mocha or
AVA test is not mistaken for one that never checks anything.

### Invocation { #invocation }

```bash
npx falsegreen-js                 # scan cwd
npx falsegreen-js src test        # scan paths
npx falsegreen-js --staged        # only test files staged in git (pre-commit)
```

Each finding is reported with its [pyramid level](../concepts/pyramid.md) (unit / integration /
e2e, read from the file's imports) and a one-line fix hint; the summary breaks findings down by
level and lists the most common fixes.

### Output formats { #output-formats }

`--format text|json|sarif|junit` selects the report shape (default `text`). `--json` stays as an
alias for `--format json`. These match the [Python sibling](python.md) byte-for-concept, so a
pipeline can swap one scanner for the other.

```bash
npx falsegreen-js . --json                # machine-readable JSON
npx falsegreen-js . --format sarif        # SARIF 2.1.0
npx falsegreen-js . --format junit        # JUnit XML
npx falsegreen-js . --output report.json  # write to a file
npx falsegreen-js . --output .falsegreen/ # write report.<ext> into a directory
```

- **`sarif`** emits SARIF 2.1.0: one rule per code present, one result per finding, `error` for
  HIGH, `warning` for LOW, `note` for off. Result tags carry the judgment (J1-J6), the risk group
  (`risk:effectiveness`...), and the level. Upload it to GitHub code scanning to see findings
  inline on the PR.
- **`junit`** emits JUnit XML: HIGH findings become `<failure>`, everything else `<skipped>`, so a
  CI test reporter surfaces them as a failing suite.

The JSON report also carries `riskGroup` and `oracleRegistryVersion`: codes are classified into
closed risk groups, and the oracle registry (sync-fail / promise / runner-registered / value-only)
is versioned so async detection is principled rather than string-matched. `--output` takes a file
or a directory; keep the output directory gitignored.

### Configuration { #configuration }

**Disable / enable codes (CLI).**

```bash
npx falsegreen-js --disable C7,JS3   # turn specific codes off
npx falsegreen-js --enable D8,M2     # re-activate off/opt-in codes at catalog severity
npx falsegreen-js --diagnostics      # include the D*/M* maintainability group as warnings
```

`--enable` flips a default-off code on at its catalog severity; it cannot raise a code above
catalog. A code passed to both `--enable` and `--disable` stays off, `--disable` wins.

**Diagnostics group (default off).** These are not false-green, so they warn only: `D1` assertion
roulette, `D3` duplicate assert, `D4` index-only `each` cases, `D6` `console.*` in a test, `D7`
anonymous test, `D8` magic number, `M2` long test body. Enable all with `--diagnostics`, or per
code via config `severity`.

**Inline suppression.** A comment on the offending line silences a justified finding:

```ts
expect(user.id).toBe(user.id); // falsegreen: ignore[C7]
expect(x);                     // falsegreen: ignore
```

**Project config file.** `falsegreen.json`, `.falsegreenrc.json`, or a `"falsegreen"` key in
`package.json`:

```json
{
  "disable": ["C8"],
  "exclude": ["**/legacy/**"],
  "severity": { "JS3": "off", "C16": "high" }
}
```

Precedence, highest first: CLI `--disable`, CLI `--enable`, config `disable` / `severity`, catalog
default.

**Config audit.** `--config-audit` is a separate mode. Instead of scanning test files, it reads
the Jest / Vitest config (`package.json` `jest` field, `jest.config.*`, `vitest.config.*`) and
reports the project-layer ways a suite stays green by configuration:

- `PL10` - `passWithNoTests` passes an empty or filtered-to-nothing run.
- `PL7` - no `coverageThreshold` / `coverage.thresholds`.
- `PL8` - `bail` stops the run early.

**Baseline (ratchet).** Adopt the scanner on a large codebase without fixing every legacy finding
at once:

```bash
npx falsegreen-js --write-baseline   # record current findings to .falsegreen-baseline.json, exit 0
npx falsegreen-js --baseline         # report and fail only on findings not in the baseline
```

`--baseline [PATH]` and `--write-baseline [PATH]` default to `.falsegreen-baseline.json`. A
finding's identity is a content fingerprint (sha1 of relative path + code + detail, no line
number), so it survives unrelated line shifts. Commit the baseline, then let CI block only on
net-new findings.

### Exit codes { #exit-codes }

| Code | Meaning |
|---|---|
| `0` | clean, no findings that affect the gate |
| `10` | low-confidence findings only |
| `20` | at least one high-confidence finding |

Block the commit on `20`.

### CI integration { #ci-integration }

**GitHub Actions.** A failing job on exit `20`:

```yaml
name: falsegreen-js
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "20" }
      - run: npx falsegreen-js .   # exit 20 fails the job
```

**SARIF upload to GitHub code scanning.** Emit SARIF and hand it to the CodeQL action so findings
show inline on the pull request:

```yaml
      - run: npx falsegreen-js . --format sarif --output falsegreen.sarif
        continue-on-error: true
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: falsegreen.sarif
```

**Pre-commit hook.** Run the staged-files scan as a hook with `--staged`:

```yaml
  - repo: local
    hooks:
      - id: falsegreen-js
        name: falsegreen-js
        entry: npx falsegreen-js --staged
        language: system
        pass_filenames: false
        types_or: [javascript, ts, tsx, jsx]
```

For the layer no static scan reaches (does a green test fail when the code is wrong?), run a
mutation tester like [Stryker](https://stryker-mutator.io/). falsegreen-js is the cheap
pre-filter on every commit; the semantic cases (intent, oracle correctness) are
[falsegreen-skill](skill.md).

## What it covers

Full per-code detail in the [JS/TS catalog](../catalog/javascript-typescript.md).

| Group | Codes | Effect |
|---|---|---|
| **Shared with Python** (F1-F6) | `C2`, `C2b`, `C5`, `C6`, `C7`, `C8`, `C9`, `C16`, `C18`, `C20`, `C21`, `C23`, `C37`, `C44`, `CC` | HIGH blocks, LOW warns |
| **JavaScript-specific** | `JS1`-`JS9`, `JS11`, `JS13`, `JS15`, `JS17`, `JS18`, `JS21`, `JS22` | idem |
| **Diagnostic / coupling** (F8) | `D1`, `D3`, `D4`, `D6`, `D7`, `D8`, `M2` | opt-in |
| **Project / CI** (`--config-audit`) | `PL10` (`--passWithNoTests`), `PL7` (coverage threshold), `PL8` (bail) | reads jest/vitest config |

## Playwright (E2E and API) { #playwright }

A project that mixes `node:test`/Jest/Vitest with Playwright specs scans cleanly. Playwright
has its own assertion grammar, and reading it through the `node:test`/`assert` lens produced
false positives; the scanner now recognizes the framework and judges it on its own terms.

**How Playwright is recognized.** Three signals, any one of which marks the file:

1. the package import (`@playwright/test`, `playwright`, `playwright-core`);
2. a namespaced `test.*` API call (`test.describe`, `test.step`, `test.beforeAll`,
   `test.afterAll`) — this catches suites that import `test`/`expect` from a **fixture
   re-export** (`test.extend`/`mergeTests`, e.g. `import { test, expect } from "@company/e2e/fixtures"`),
   the pattern Playwright's own docs recommend;
3. a Playwright fixture parameter: `browserName` alone (unambiguous), or `page`/`context`/
   `request` corroborated by a Playwright-exclusive matcher such as `toBeOK`/`toHaveCount`.
   A bare `request` never flips the signal by itself, so a supertest/Express test that
   destructures `{ request }` is not mistaken for Playwright.

**What that fixes.**

- **Lifecycle hooks are not tests.** `test.beforeEach`/`afterEach`/`beforeAll`/`afterAll`,
  `test.describe`, and `test.step` are no longer analyzed as test bodies, so a setup or
  teardown hook that calls code without an assertion no longer reports `C2b` (and the
  sibling body-level codes stop misfiring on a hook). A hook is not a test; the assertion
  belongs in the `test(...)`. `test.only`/`test.serial`/`test.fail`/`test.failing` are real
  running tests and keep full body analysis.
- **Conditional skip is a runtime guard, not a disabled test.** `test.skip(condition[, reason])`,
  `test.skip()`, and the predicate form `test.skip(({ browserName }) => ...)` no longer fire
  `JS4`. Unconditional declared skips still do: `test.skip("title", fn)`, `test.skip(true, "...")`
  (a group disabled by a constant), `test.describe.skip`, `xit`/`xdescribe`/`it.todo`. The
  suppression is gated on a Playwright signal **and** a non-string first argument, so a
  `node:test`/Mocha titleless `test.skip(fn)` in a non-Playwright file still fires `JS4` — no
  false negative.
- **API-response and UI matchers count as assertions.** `expect(locator).toBeVisible()`/
  `toHaveText()`/`toHaveCount()` and `expect(apiResponse).toBeOK()` already registered as real
  oracles; this work adds the hook, skip, and file-recognition handling around them so the
  whole spec — UI or API — reads correctly.

The file-recognition signal is scoped to the skip decision only; it does not widen the pyramid
level, so a Jest/Vitest file with a bare `describe` never turns into a browser-layer file and
never starts forgiving a weak check. See [`C6`](#c6-weak-check) below.

## Playwright-safe weak-check (C6) { #c6-weak-check }

`C6` (weak check) uses a **sole-oracle** model across every runner: it fires only when a
presence/non-empty check (`toBeTruthy`/`toBeDefined`/`.length > 0`) is the enclosing test's
ONLY oracle. A weak check next to a stronger assertion in the same test (a `toBeGreaterThan(0)`
guard before a `toBe(...)`) is not flagged — the strong assertion carries the test. A test whose
assertions are all weak still reports `C6`. This matters for E2E, where a guard like
`expect(items.length).toBeGreaterThan(0)` commonly precedes the real check.

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
