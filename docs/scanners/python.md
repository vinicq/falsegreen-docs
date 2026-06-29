# falsegreen (Python)

[![CI](https://github.com/vinicq/falsegreen/actions/workflows/ci.yml/badge.svg)](https://github.com/vinicq/falsegreen/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/falsegreen.svg)](https://pypi.org/project/falsegreen/)
[![Downloads](https://img.shields.io/pypi/dm/falsegreen.svg)](https://pypistats.org/packages/falsegreen)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/vinicq/falsegreen/blob/main/LICENSE)

The deterministic Python/pytest scanner. A zero-dependency AST pass that validates each test
against the false-positive codes a parser can prove. HIGH findings block the commit, LOW ones
warn, and a diagnostic/coupling group is opt-in.

- Repository: [github.com/vinicq/falsegreen](https://github.com/vinicq/falsegreen)
- Catalog: [Python codes](../catalog/python.md)

## Install

```bash
pip install falsegreen
```

## Use

```bash
falsegreen path/to/tests        # scan
falsegreen --staged             # only staged files (pre-commit)
falsegreen --json               # machine-readable report
falsegreen --diagnostics        # include the opt-in F8 group
falsegreen --config-audit       # read pytest/coverage config for project-level false-green
```

HIGH findings exit non-zero, so the tool drops into CI and pre-commit unchanged. The report
numbers each finding with its code, judgment, pyramid level, location, evidence, and a fix hint.

## First finding

Save a test that always passes:

```python
# test_demo.py
def test_demo():
    assert True
```

Run the scanner over it:

```bash
falsegreen test_demo.py
```

It reports:

```
test_demo.py:3  [C5] always-true check (assert True / tuple / or True)
    level: unit   fix: assert the real behaviour, not a constant or tautology

Summary: 1 high, 0 low.
```

## Reading a finding

Each line carries the same fields:

- `test_demo.py:3` - the file and the line that triggered it.
- `[C5]` - the catalog code. `C5` is the always-true check. Every code is
  explained in the [Python catalog](../catalog/python.md).
- `level: unit` - which level of the [test pyramid](../concepts/pyramid.md) the
  file sits at; it changes what counts as a real check.
- `fix:` - a one-line hint. Here: assert the real behaviour, not a constant.

Exit codes wire it into CI: `0` clean, `10` low-confidence only, `20` at least
one high-confidence finding. Block the build on `20`.

## What it covers

The most complete scanner of the family - it is the reference the others mirror. The full
per-code detail is in the [Python catalog](../catalog/python.md).

| Group | Codes | Effect |
|---|---|---|
| **False-positive** (F1-F6) | ~45 active `C*` codes + `CC` | HIGH blocks, LOW warns |
| **Diagnostic / coupling** (F8) | `D1`, `D3`, `D4`, `D5`, `D6`, `M2` | opt-in, never blocks |
| **Project / CI** (F5, `--config-audit`) | `PL2` (filterwarnings not error), `PL7` (no `--cov-fail-under`), `PL8` (`-x`/`--maxfail` masks the count) | reads config, reports |

## What it does not cover, and why

### Out of scope (the wrong axis)
Brittleness/false-red, hygiene, slow, design, naming, duplication, runtime/culture are not
false-green. See [coverage vs the literature](../concepts/denominator.md) for the full boundary.

### Codes deliberately not implemented
These were evaluated against the consolidated catalog and left out, each for a reason. Leaving
them out is the precision-first policy: a false positive is worse than a miss.

| Code | What it would flag | Why not |
|---|---|---|
| **C40** | `assert mock.attr` with no spec (always truthy) | without spec analysis the false-positive rate is high; the concept lives in the skill (F7) |
| **C46** | real network/DB with no double (`requests`, `socket`) | legitimate in an integration test; flagging it needs to know the level, so it routes to the skill / `--config-audit` |
| **C47** | assertion depends on dict/set ordering | high false positive (most collections are deterministic in use); a skill note instead |

### Reserved for the semantic pass (F7)
Mocking the unit under test (case 10), asserting the value fed to the mock (case 11),
re-implementing the production formula (case 12), an expected value that contradicts intent
(case 18), borrowed shared state (case 15). No AST proves intent or inter-procedural flow. These
live in [falsegreen-skill](skill.md). `C14` (snapshot of the code's own output) is the only
codifiable corner.

### Needs runtime (not promised statically)
`python -O` stripping `assert`, a collection error reported as "0 tests passed", a CI step that
runs a subset and reports green (`PL1`/`PL4`/`PL6`). `PL1` now has a config-discoverable slice:
`--config-audit` flags `python -O`/`-OO` or `PYTHONOPTIMIZE=1` set in `tox.ini`/pytest `addopts`
as a project-level warning. The rest only appear when the suite runs; they are documented, not
claimed. The honest path is mutation testing (mutmut, cosmic-ray), which is out of band.

See [scope and honesty](../concepts/scope.md) for the layer boundary.
