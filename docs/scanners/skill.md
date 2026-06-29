# falsegreen-skill (semantic LLM pass)

[![CI](https://github.com/vinicq/falsegreen-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/vinicq/falsegreen-skill/actions/workflows/ci.yml)
[![npm](https://img.shields.io/npm/v/falsegreen-skill.svg)](https://www.npmjs.com/package/falsegreen-skill)
[![Downloads](https://img.shields.io/npm/dm/falsegreen-skill.svg)](https://www.npmjs.com/package/falsegreen-skill)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/vinicq/falsegreen-skill/blob/master/LICENSE)

The semantic layer and a superset of the three static scanners. It reads a test against its
intent, the spec, and the production code to catch the false-greens no parser sees (F4, F7), and
it carries every structural code of the scanners plus the AI-only S-series.

- Repository: [github.com/vinicq/falsegreen-skill](https://github.com/vinicq/falsegreen-skill)
- Catalog: [semantic codes](../catalog/semantic.md)

## What it is

Not Claude-specific. The same protocol is packaged for Claude Code, Codex, Gemini, Cursor, plain
LLM prompts, API usage, and an npm CLI. For Python it applies the complete falsegreen catalog
directly; for TypeScript, JavaScript, and Robot Framework it is the primary detection tool.

## The protocol (J1-J6)

Every test is read through six [judgments](../concepts/judgments.md): does the assertion run, is
the expected value from an [independent oracle](../concepts/oracle.md), is the real unit
exercised, is the assertion sufficient, is it free of coupling to internals, does it pass in
isolation. A false positive is worse than a miss, so a wrong-value finding is not reported without
citing an oracle.

## Install and first run

The skill runs on several hosts (Claude Code, Codex, Gemini, Cursor) and as a
standalone npm CLI. The CLI needs only Node 18+ and an API key for the provider
you choose:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
npx falsegreen-skill analyze tests/test_demo.py
```

`--provider openai` or `--provider gemini` switches the model; `--json` and
`--fail-on-high` wire it into CI. Per-host setup (the Claude Code plugin, the
Codex and Gemini extensions, the Cursor rule) is in the
[skill README](https://github.com/vinicq/falsegreen-skill#installation).

## First finding

Given a test that asserts the mock back to itself:

```python
# tests/test_tax.py
def test_calculate_tax(mock_calc):
    mock_calc.return_value = 0.15
    result = calculate_tax(100, mock_calc)
    assert result == mock_calc.return_value
```

`npx falsegreen-skill analyze tests/test_tax.py` reports:

```
CASE 11 (J2) - HIGH - Python - unit - behavior

Test: test_calculate_tax (line 2-5)
Finding: The assertion checks mock_calc.return_value - the same value the mock
was configured to return. It passes for any result, including a wrong one.
Evidence:
  mock_calc.return_value = 0.15
  assert result == mock_calc.return_value
Fix hint: Assert against an independently computed value, e.g. assert result == 15.0.
```

## Reading a finding

Each finding names five things:

- `CASE 11` - the [semantic code](../catalog/semantic.md). The skill also emits
  the structural `C*`, `JS*`, and `R*` codes the scanners use.
- `(J2)` - the failed [judgment](../concepts/judgments.md): the expected value is
  not from an independent [oracle](../concepts/oracle.md).
- `HIGH` - confidence. HIGH means no plausible legitimate reading; LOW warns.
- `Python - unit - behavior` - language, [pyramid level](../concepts/pyramid.md),
  and test intent.
- `Finding` / `Evidence` / `Fix hint` - what is wrong, the lines that prove it,
  and how to repair it.

## What it covers

The broadest tool of the family. It is the superset, so its coverage is the union of everything
the scanners catch plus what only a reader of intent can:

| Layer | Coverage |
|---|---|
| **All structural codes** | every `C*` (Python), `JS*`, and `R*` code from the three scanners, applied by reading the source |
| **Semantic cases** | 10 (mocks the unit), 11 (asserts the stub), 12 (re-implements the formula), 15 (shared state), 18 (contradicts the spec) |
| **The S-series** (AI-only) | `S1`-`S13`: intent mismatch, irrelevant oracle, plausible-but-wrong value, coarse oracle, tests the framework, happy-path-only, value lifted from output, mock through indirection, self-fulfilling arrangement, asserts the log, negative-only security check, patches core logic, cross-file order dependence |
| **DSL passes** | Gherkin `.feature` and Tavern `*.tavern.yaml` (see [Gherkin and Tavern](../catalog/gherkin-tavern.md)) |
| **Level awareness** | reads unit / integration / E2E from signals and adjusts the oracle |

## Modes

- **Detect** - read a suite and report findings (J1-J6, level, evidence, fix hint).
- **Author** - generate tests that are not false-green by construction, one spec per pyramid
  level.
- **AI-fix gate (F7)** - propose a strengthened test and validate it with a bidirectional
  mutation gate (pass on clean code, fail on the reintroduced bug).

## What it does not cover, and why

### The runtime half of F7
The skill proposes a strengthened test and self-checks that it *can* fail, but it does not run
mutation testing - that is the host's job (mutmut, cosmic-ray, Stryker). A strengthened test is
only *accepted* after the bidirectional gate runs, and the skill never invokes the mutation tool.
So the live gate result is out of the skill's hands by design.

### The wrong axis
Even as the broadest tool, it stays false-green only. Brittleness/false-red, pure hygiene, slow,
design, naming, and runtime-culture smells are out, the same boundary as the scanners. See
[coverage vs the literature](../concepts/denominator.md).

### Determinism trade-off
The semantic findings are operator-confirmed, not deterministic: confidence is LOW or HIGH by how
clear the contradiction is, and the skill shows its reasoning instead of auto-blocking. Where a
parser can prove a pattern, the [static scanners](python.md) are the faster, deterministic
pre-filter; the skill is the complete multi-stack net. See
[scope and honesty](../concepts/scope.md).
