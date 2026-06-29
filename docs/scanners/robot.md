# robotframework-falsegreen (Robot Framework)

[![CI](https://github.com/vinicq/robotframework-falsegreen/actions/workflows/ci.yml/badge.svg)](https://github.com/vinicq/robotframework-falsegreen/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/robotframework-falsegreen.svg)](https://pypi.org/project/robotframework-falsegreen/)
[![Downloads](https://img.shields.io/pypi/dm/robotframework-falsegreen.svg)](https://pypistats.org/packages/robotframework-falsegreen)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/vinicq/robotframework-falsegreen/blob/main/LICENSE)

The deterministic Robot Framework scanner. A static scan over the official parser
(`robot.api.get_model`), no execution. It recognizes the verification vocabulary across the Robot
library ecosystem, so a real check is not mistaken for "no oracle".

- Repository: [github.com/vinicq/robotframework-falsegreen](https://github.com/vinicq/robotframework-falsegreen)
- Catalog: [Robot Framework codes](../catalog/robot.md)
- CLI command: `rffalsegreen`

## Install

```bash
pip install robotframework-falsegreen
```

## Use

```bash
rffalsegreen path/to/suite              # scan
rffalsegreen --format json|sarif|junit  # output shape
rffalsegreen --config-audit             # robot.toml / invocation for project-level checks
```

## First finding

Save a suite whose only step is a no-op:

```robotframework
*** Test Cases ***
Login Works
    No Operation
```

Run the scanner over it:

```bash
rffalsegreen demo.robot
```

It reports:

```
demo.robot:3  [R4] No Operation as the only step - the test verifies nothing
    level: e2e   fix: call a real verification keyword (Should Be Equal, status check)

Summary: 1 high, 0 low.
```

## Reading a finding

Each line carries the same fields:

- `demo.robot:3` - the file and the line that triggered it.
- `[R4]` - the catalog code. `R4` is a `No Operation`-only test. Every code is
  explained in the [Robot catalog](../catalog/robot.md).
- `level: e2e` - which level of the [test pyramid](../concepts/pyramid.md) the
  suite sits at, read from the imported library in `*** Settings ***`.
- `fix:` - a one-line hint. Here: call a real verification keyword.

`--format json|sarif|junit` gives a machine-readable report for CI.

## Complete usage and configuration { #complete-usage-and-configuration }

The getting-started above is the five-minute path. This section is the full reference:
every install channel, every output format, every configuration knob, the exit-code
contract, and the CI wiring. It mirrors what the [project README](https://github.com/vinicq/robotframework-falsegreen)
documents.

### Install channels { #install-channels }

```bash
pip install robotframework-falsegreen   # CLI command: rffalsegreen
```

The scanner runs a static pass over the official Robot parser (`robot.api.get_model`); it never
executes a suite, and it scopes to `.robot` / `.resource` files.

### Invocation { #invocation }

```bash
rffalsegreen                  # scan cwd
rffalsegreen tests/           # scan a path
rffalsegreen --disable C16    # turn off specific codes
```

Each finding is reported with its [pyramid level](../concepts/pyramid.md) (unit / integration /
e2e, read from the suite's imported libraries) and a one-line fix hint; the text summary breaks
findings down by level and lists the most common fixes.

### Output formats { #output-formats }

`--format text|json|sarif|junit` selects the report shape (default `text`). `--json` stays as an
alias for `--format json`.

```bash
rffalsegreen --format json            # machine-readable JSON
rffalsegreen --format sarif           # SARIF 2.1.0
rffalsegreen --format junit           # JUnit XML
rffalsegreen --output report.sarif    # write to a file
rffalsegreen --output .falsegreen/    # write report.<ext> into a directory
```

- **`sarif`** emits SARIF 2.1.0 (tool name `robotframework-falsegreen`): HIGH maps to `error`,
  LOW to `warning`, off/info to `note`, and each result is tagged with its judgment family and
  pyramid level so GitHub code scanning can group and filter findings.
- **`junit`** emits JUnit XML: one testcase per finding, HIGH becomes `<failure>`, anything lower
  `<skipped>`.

`--json` keeps its envelope (`tool` / `version` / `judgments` / `findings`). `--output` takes a
file or a directory; an extension-less or trailing-slash path receives `report.<ext>` (for JUnit,
`report.xml`). Keep the output directory gitignored.

The verification recognizer knows the `Should` convention plus library forms (SeleniumLibrary,
the Browser assertion engine, RequestsLibrary including `expected_status=<code>`, RESTinstance,
DatabaseLibrary), so an API or UI assertion is not misread as no-oracle.

### Configuration { #configuration }

**Disable codes (CLI).** `--disable C16` turns specific codes off for one run.

**Inline suppression.** A comment on the offending line silences a justified finding without
disabling the code suite-wide. The token and bracket syntax match falsegreen (Python) and
falsegreen-js:

```robotframework
*** Test Cases ***
Polls A Real Service
    Sleep    1s    # falsegreen: ignore[C16]      # silence only C16 on this line
    Should Be Equal    ${result}    ${expected}
```

`# falsegreen: ignore` (no brackets) silences every code on that line; `ignore[C16,C20]` silences
only the listed codes. Only the exact `falsegreen:` token suppresses; a plain `# ignore` does not.

**Diagnostics group (default off).** The maintainability codes (`D2`, `M2`) are not false-green,
so they are opt-in; enable them per code via config `severity`.

**Config audit.** `--config-audit` is a separate mode. Instead of scanning suites, it reads the
Robot run config (`robot.toml`, `pyproject.toml` `[tool.robot]`, and `*.args` argument files found
recursively, skipping ignored directories like `results/` / `output/`) and reports `PL9`: a
`--skiponfailure` / `--noncritical` option that turns a failing test into a non-fatal pass (legacy,
removed in RF 4+). The per-file scan cannot see run config.

**Baseline (adopt on a suite that already has findings).** Record a baseline and then fail only on
new findings:

```bash
rffalsegreen --write-baseline    # record current findings to .falsegreen-baseline.json
rffalsegreen --baseline          # scan, suppressing everything in the baseline
```

Both flags take an optional path (default `.falsegreen-baseline.json`). The baseline fingerprints a
finding by content (sha1 of relative path, code, test/keyword name, detail, no line number), so it
survives edits that shift a test up or down the file. Commit the baseline so CI sees the same set.

### Exit codes { #exit-codes }

| Code | Meaning |
|---|---|
| `0` | clean, no findings that affect the gate |
| `10` | low-confidence findings only |
| `20` | at least one high-confidence finding |

Wire exit `20` into CI to block the merge.

### CI integration { #ci-integration }

**GitHub Actions.** A failing job on exit `20`:

```yaml
name: robotframework-falsegreen
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.x" }
      - run: pip install robotframework-falsegreen
      - run: rffalsegreen tests/   # exit 20 fails the job
```

**SARIF upload to GitHub code scanning.** Emit SARIF and hand it to the CodeQL action so findings
show inline on the pull request:

```yaml
      - run: rffalsegreen tests/ --format sarif --output falsegreen.sarif
        continue-on-error: true
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: falsegreen.sarif
```

**Pre-commit hook.** Add to `.pre-commit-config.yaml`:

```yaml
  - repo: https://github.com/vinicq/robotframework-falsegreen
    rev: v0.3.0
    hooks:
      - id: rffalsegreen
```

The hook scopes to `.robot` / `.resource` files and passes the staged paths to the scanner, so it
never re-scans `results/` / `output/`. It honors the exit codes, so a HIGH finding fails the
commit. Run on demand against the staged files with `pre-commit run rffalsegreen`.

Robot has no standard mutation tester, so the runtime layer (does a green test fail when the code
is wrong?) is manual review; the intent-level cases are [falsegreen-skill](skill.md).

## What it covers

Full per-code detail in the [Robot catalog](../catalog/robot.md).

| Group | Codes | Effect |
|---|---|---|
| **Shared with Python** (F1-F6) | `C2`, `C2b`, `C3`, `C5`, `C6`, `C7`, `C9`, `C16`, `C20`, `C21`, `C23`, `C32`, `C37`, `CC` | HIGH blocks, LOW warns |
| **Robot-specific** | `R1` (Pass Execution), `R2` (hollow verifier), `R3` (test cases in .resource), `R4` (No Operation), `R5` (empty template), `R6` (Should Be True on a string), `R7` (hollow template keyword) | idem |
| **Diagnostic** (F8) | `D2`, `M2` | opt-in |
| **Project / CI** (`--config-audit`) | `PL9` (legacy `--skiponfailure`/noncritical via robot.toml/args) | reads config |

The verification recognizer knows the `Should` convention plus library forms (SeleniumLibrary,
Browser assertion engine, RequestsLibrary including `expected_status=<code>`, RESTinstance,
DatabaseLibrary), so an API or UI assertion is not misread as no-oracle.

## What it does not cover, and why

### Out of scope (the wrong axis)
Same boundary as the family. See [coverage vs the literature](../concepts/denominator.md).

### Deliberately not implemented
| What | Why not |
|---|---|
| **`Wait Until Keyword Succeeds`** as a flaky mask (catalog RF16) | legitimate retry around genuine E2E flakiness; high false positive |
| **Cross-test shared state** (`Set Suite/Global Variable` read by a sibling) | needs whole-suite ordering analysis, not a per-file fact; the HowTo guide even sanctions some inter-test dependency. High false positive |
| **`C31` - captured value never used** (`${x}= Get Text` never read) | a real recall gap, but the false-positive surface (value used only in `Log`, in an `Evaluate` string, or in teardown) needs careful bounding. Deferred to a second pass ([issue #34](https://github.com/vinicq/robotframework-falsegreen/issues/34)) |
| **Dead user keyword** (catalog RF6) | needs a cross-file project pass; the scanner is single-file today |
| **Pure hygiene** (bad naming, comment rate, long parameter lists) | Robocop owns the Robot style guide; the scanner does not duplicate it |

### No clean Robot equivalent
A few Python/JS sibling codes have no faithful Robot form and are intentionally skipped rather
than forced. The catalog notes them explicitly.

### Beyond the scanner
Runtime smells (Test Run War, cross-suite order dependence) need the suite to run; the semantic
slice is [falsegreen-skill](skill.md). See [scope and honesty](../concepts/scope.md).
