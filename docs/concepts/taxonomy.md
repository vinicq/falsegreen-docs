# Failure taxonomy (F1-F8)

Every code in the catalog maps to a failure mode. The taxonomy is the conceptual axis: it
answers *how* a test goes green without protecting anything, independent of language. The
product policy (block, warn, or diagnostic) is a separate axis, decided per code.

| Family | Failure mode | Risk axis | Layer that resolves it |
|---|---|---|---|
| **F1** | Checks nothing (no oracle) | effectiveness | static, per file |
| **F2** | The check exists but never runs | execution | static, per file |
| **F3** | The check is trivial (always passes) | effectiveness | static, per file |
| **F4** | Checks the wrong thing | effectiveness (oracle) | static (partial) + skill |
| **F5** | The test drops out of the count (skip / not collected) | execution | static + project layer |
| **F6** | Passes or fails by luck (non-determinism) | non-determinism | static (proxy) + runtime |
| **F7** | Circular or semantic oracle | semantic | skill + mutation testing |
| **F8** | Hygiene / readability (not false-green) | structure | diagnostic opt-in / linter |

## How to read it

**F1-F6 are false-green.** The test reports success while the code it covers may be broken.
These block or warn, depending on confidence.

**F7 is semantic.** The oracle is circular (the test re-derives the expected value from the
code, or mocks the unit it claims to test). No parser proves intent. The
[skill](../scanners/skill.md) reads it; a live gate with mutation testing confirms it.

**F8 is not false-green.** The test still protects; it is just hard to read or maintain
(assertion roulette, an over-long body, a magic number). These are diagnostic, off by default,
and dedicated linters (ruff, ESLint, Robocop) also cover them. Where a linter covers it, the
scanners defer.

## Why a separate axis from the product groups

A scanner sorts its output into three CLI groups: **false-positive** (blocks or warns),
**diagnostic** (opt-in), and **coupling**. That is the *policy*. F1-F8 is the *map*. The two
do not collide: F1-F6 land in the false-positive group, F7 routes to the skill and mutation
testing, F8 is the diagnostic group.

The taxonomy also names the denominator for measurement. When the catalog is cross-walked
against the published test-smell literature, almost every external smell maps onto an F1-F8
mode, which keeps the academic claims honest: the tools measure against a named set of failure
modes, not an open-ended list.

## The two layers a parser cannot reach

A clean file is not a clean suite. Two failure modes survive a per-file scan:

- **F5 at the project level.** The file asserts correctly, but the runner is configured to let
  an empty or partial run pass: `--passWithNoTests`, a coverage gate that is never enforced, a
  `filterwarnings` that never becomes an error. The `--config-audit` mode reads the project and
  CI config to catch these.
- **F7 at the semantic level.** The assertion runs and looks specific, but the expected value
  comes from the code itself, or the mock stands in for the unit under test. This needs reading
  the test against its intent, which is the skill, and proving it with mutation, which is
  runtime.

See [judgments (J1-J6)](judgments.md) for the per-test questions that decide which family a
finding belongs to.
