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

## Complete usage and configuration { #complete-usage-and-configuration }

The getting-started above is the five-minute path. This section is the full reference:
every install channel, every output format, every configuration knob, the exit-code
contract, and the CI wiring. It mirrors what the [project README](https://github.com/vinicq/falsegreen)
documents.

### Install channels { #install-channels }

Python 3.8 or newer, no third-party runtime dependencies.

```bash
pip install falsegreen          # project or virtualenv install
uvx falsegreen tests/           # run once without installing (uv)
pipx run falsegreen tests/      # run once without installing (pipx)
```

`python -m falsegreen ...` is equivalent to the `falsegreen` command when the entry point
is not on `PATH`.

### Invocation { #invocation }

```bash
falsegreen                        # scan the current directory
falsegreen tests/                 # scan a folder or a single file
falsegreen --staged               # only the test files staged in git (pre-commit)
falsegreen --summary              # one-line "N scanned, M flagged" to stderr
```

The scanner reads the test files only, it never imports or runs them, so a broken or hostile
test cannot execute through it. Each finding carries its [pyramid level](../concepts/pyramid.md)
(unit / integration / e2e, read from the file's imports) and a one-line fix hint; the text
summary breaks findings down by level and lists the most common fixes.

### Output formats { #output-formats }

`--format text|json|sarif|junit` selects the report shape (default `text`). `--json` stays as
an alias for `--format json`.

```bash
falsegreen tests/ --json                  # machine-readable JSON
falsegreen tests/ --format sarif          # SARIF 2.1.0
falsegreen tests/ --format junit          # JUnit XML
falsegreen tests/ --output report.sarif   # write to a file
falsegreen tests/ --output .falsegreen/   # write report.<ext> into a directory
```

- **`json`** carries the full envelope: tool, version, judgments, and the findings list.
- **`sarif`** emits SARIF 2.1.0, mapping HIGH to `error` and LOW to `warning`, for GitHub code
  scanning and inline pull-request annotations.
- **`junit`** emits JUnit XML, where HIGH findings become `<failure>` so a CI test reporter
  surfaces them as a failing suite.

`--output` takes a file or a directory: an extension-less or trailing-slash path (`.falsegreen/`)
receives `report.<ext>` for the chosen format. Reports are run artifacts, so keep the output
directory gitignored.

### Configuration { #configuration }

**Disable codes (CLI).** `--disable C6,C2b` turns specific codes off for one run.

**Inline suppression.** A comment on the offending line silences a justified finding without
disabling the code suite-wide:

```python
assert user.id == user.id  # falsegreen: ignore[C7]   silence only C7 on this line
assert x                   # falsegreen: ignore        silence every code on this line
```

Only the exact `falsegreen:` token suppresses; a plain `# ignore` does not.

**Project config file.** `[tool.falsegreen]` in `pyproject.toml`, or a flat `.falsegreen.toml`
at the repo root (`.falsegreen.toml` wins if both exist). Point at a specific file with
`--config PATH`.

```toml
[tool.falsegreen]
disable = ["C13b"]            # turn these codes off everywhere
exclude = ["tests/legacy/*"]  # skip files matching these globs
long_test_threshold = 30      # line-count limit for M2 (default: 50)
inline_setup_threshold = 3    # statement limit for D5 (default: 5)

[tool.falsegreen.severity]
C8 = "high"    # promote: now blocks the commit (exit 20)
C6 = "off"     # same as adding C6 to disable
C22 = "low"    # enable the async-never-awaits check
D1 = "info"    # enable Assertion Roulette (diagnostic, never blocks)
M2 = "info"    # enable Long Test Method (diagnostic)
```

`severity` values: `high`, `low`, `info`, or `off`. `info` findings appear in the
DIAGNOSTIC / COUPLING sections and do not affect the exit code, the opt-in F8 group. The
`long_test_threshold` and `inline_setup_threshold` keys live directly under
`[tool.falsegreen]`, not inside `[severity]`. Precedence, highest first: `--disable` on the
CLI, inline `# falsegreen: ignore`, the config file, the built-in default.

**Config audit.** `--config-audit` is a separate mode. Instead of scanning test files, it
reads the project's pytest and coverage config (`pyproject.toml`, `pytest.ini`, `tox.ini`,
`setup.cfg`) and reports the project-layer ways a suite stays green by configuration:

- `PL1` - `python -O` / `PYTHONOPTIMIZE` strips every `assert` at runtime.
- `PL2` - `filterwarnings` does not promote warnings to errors.
- `PL7` - no coverage gate (`--cov-fail-under` absent).
- `PL8` - `addopts` stops the run early with `-x` / `--maxfail`, masking the count.

**Baseline (adopt on a legacy repo).** Record the findings you already have, then fail only
on new ones:

```bash
falsegreen --write-baseline tests/   # write .falsegreen-baseline.json, exit 0
falsegreen --baseline tests/         # suppress recorded findings, fail on new ones
```

A finding is fingerprinted by relative path, code, detail, and normalized source line (not
line number), so prepending code does not re-trigger a baselined finding. Commit
`.falsegreen-baseline.json` and the ratchet only tightens.

### Exit codes { #exit-codes }

| Code | Meaning |
|---|---|
| `0` | clean, no findings that affect the gate |
| `10` | low-confidence findings only |
| `20` | at least one high-confidence finding |

Block the build on `20`. The pre-commit hook honors the same contract.

### CI integration { #ci-integration }

**GitHub Actions.** A failing job on exit `20`:

```yaml
name: falsegreen
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.x" }
      - run: pip install falsegreen
      - run: falsegreen tests/   # exit 20 fails the job
```

**SARIF upload to GitHub code scanning.** Emit SARIF and hand it to the CodeQL action so
findings show inline on the pull request:

```yaml
      - run: falsegreen tests/ --format sarif --output falsegreen.sarif
        continue-on-error: true
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: falsegreen.sarif
```

**Pre-commit hook.** Add to `.pre-commit-config.yaml`:

```yaml
  - repo: https://github.com/vinicq/falsegreen
    rev: v0.6.0
    hooks:
      - id: falsegreen
```

Then `pre-commit install`. The hook entry is `falsegreen --staged` with `pass_filenames: false`,
so it reads the staged test files itself; do not add file arguments or re-enable `pass_filenames`,
or some files scan twice. Pin a tag (never a branch) so local runs and CI use the same scanner;
`pre-commit autoupdate` rewrites the `rev`. Bypass once with `git commit --no-verify`, or set
`FALSEGREEN_BLOCK=0` to make the hook warn-only. To run on push instead of commit, set
`stages: [pre-push]` under the hook. A raw git hook without the framework:

```bash
python -m falsegreen.hook_install --repo .      # install
python -m falsegreen.hook_install --uninstall   # remove
```

For the semantic cases a parser cannot reach, pair the scanner with the
[falsegreen-skill](skill.md) LLM pass.

## What it covers

The most complete scanner of the family - it is the reference the others mirror. The full
per-code detail is in the [Python catalog](../catalog/python.md).

| Group | Codes | Effect |
|---|---|---|
| **False-positive** (F1-F6) | ~45 active `C*` codes + `CC` | HIGH blocks, LOW warns |
| **Diagnostic / coupling** (F8) | `D1`, `D3`, `D4`, `D5`, `D6`, `M2` | opt-in, never blocks |
| **Project / CI** (F5, `--config-audit`) | `PL2` (filterwarnings not error), `PL7` (no `--cov-fail-under`), `PL8` (`-x`/`--maxfail` masks the count) | reads config, reports |

## Playwright and API tests (C6) { #playwright-api }

Playwright for Python covers two kinds of test: E2E of the UI (`page`, locators,
`expect(locator).to_be_visible()`) and API tests (`APIRequestContext`, `assert response.ok`,
`expect(api_response).to_be_ok()`). The scanner already recognizes both — `expect(...)` counts
as an assertion, `playwright`/`pytest_playwright` raise the file to the e2e/browser level, and
a fixture with `autouse=True` that only sets up is not analyzed as a test.

The one refinement worth calling out is `C6` (weak check) over the API idiom. `assert <resp>.ok`
is the canonical way a Playwright API test asserts a 2xx (the same `.ok` property exists on
`requests.Response` and Selenium's `APIResponse`). The web/UI softening used to key the presence
check on the response variable's **root name**, so `assert response.ok` was quiet but
`assert new_issue.ok` — the exact form in the official Playwright API-testing docs — fired a
false positive. The softening now anchors on the `.ok` **attribute form**, not the variable name:

- `assert new_issue.ok` / `assert issues.ok` in a Playwright/`requests` test no longer fires `C6`;
- the gate stays scoped to web/browser context — an `assert x.ok` at unit level still fires `C6`;
- it is anchored on the `ast.Attribute` node, so a bare `assert ok` (a plain name, a legitimate
  weak guard) and the `.ok()` call form (`.ok` is a property, not callable) keep firing `C6`.

Known, bounded give-back: `assert m.ok` on a spec-less `Mock()` in a web test is softened too
(the auto-created attribute is truthy). This already applied to `assert response.ok`; the fix
only widens it to any response name. Accepted under precision-over-recall — a false positive that
blocks a real API test is worse than this miss. Unlike [falsegreen-js](javascript.md#c6-weak-check),
which suppresses `C6` only when the weak check is the sole oracle, the Python scanner softens
`.ok` per occurrence; the py/js behaviour gap is intentional.

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
