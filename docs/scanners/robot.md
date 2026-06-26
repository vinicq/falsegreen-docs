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
