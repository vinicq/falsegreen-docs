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
  level. Now on the CLI too, via `generate`, not only host-side.
- **AI-fix gate (F7)** - propose a strengthened test and validate it with a bidirectional
  mutation gate (pass on clean code, fail on the reintroduced bug).

## Complete usage and configuration { #complete-usage-and-configuration }

The first-run above is the five-minute path. This section is the full reference: the CLI
(`analyze`, `generate`, and the `fix` mutation-gate mode), provider configuration for every supported
backend, and the per-host enable steps. It mirrors what the [project README](https://github.com/vinicq/falsegreen-skill),
[docs/cli.md](https://github.com/vinicq/falsegreen-skill/blob/master/docs/cli.md), and
[providers.md](https://github.com/vinicq/falsegreen-skill/blob/master/providers.md) document.

### The CLI { #the-cli }

Node 18 or newer, zero dependencies. Install or run on demand:

```bash
npm install -g falsegreen-skill
npx falsegreen-skill analyze tests/test_payment.py
```

Three commands:

```text
falsegreen-skill analyze <file...> [options]
falsegreen-skill generate <spec-file> [--lang <language>] [options]
falsegreen-skill fix <test-file> --case <code> --line <n> [options]
```

The CLI sends each file to an LLM provider with the J1-J6 protocol as the system prompt and
prints the findings report. It identifies the language from the file extension, so Python,
TypeScript, and JavaScript work the same way with no extra flag.

#### `analyze` flags { #analyze-flags }

```bash
npx falsegreen-skill analyze tests/test_orders.py                 # single file
npx falsegreen-skill analyze tests/test_orders.py tests/test_pay.py  # multiple (separate calls)
npx falsegreen-skill analyze tests/ --json --fail-on-high          # CI gate: exits 2 on a HIGH finding
npx falsegreen-skill analyze tests/ --model claude-opus-4-8        # deeper model for case 18
npx falsegreen-skill analyze tests/ --temperature 0.0              # more deterministic (default 0.2)
```

| Flag | Description | Default |
|---|---|---|
| `--provider <name>` | `anthropic`, `openai`, `gemini`, or `openai-compatible` | `anthropic` |
| `--model <model>` | Model override. Required for `openai-compatible` | per provider (below) |
| `--base-url <url>` | API base URL. Required for `openai-compatible` | none |
| `--json` | Validate and output JSON conforming to `schema/report.json` | off |
| `--conventions <file>` | Conventions YAML/text block injected per SKILL.md Step 0 | none |
| `--temperature <n>` | Sampling temperature 0.0-1.0. Skipped for OpenAI o-series (o3, o4-mini) | `0.2` |
| `--max-tokens <n>` | Max output tokens per request | `4096` |
| `--fail-on-high` | Exit 2 when any HIGH finding is present. Requires `--json` | off |

With `--json`, each model response is validated against the canonical schema and emitted as one
aggregate report. If your project uses custom assertion helpers or intentional patterns that look
like smells, declare them in a conventions file and pass `--conventions`; the block is injected
before the file content, per Step 0 of the protocol, so the model reads it before judging.

#### `fix` mode (mutation gate) { #fix-mode }

`analyze` finds a false-green; `fix` proposes a stronger test and proves it before you trust it.
It is opt-in, Python/pytest only, and propose-only: it prints a test-file patch but never applies
it and never edits production code.

```bash
# propose a patch for a C2b finding and run the gate against the real SUT
npx falsegreen-skill fix tests/test_discount.py --case C2b --line 14 --sut src/discount.py

# parse + preserve only, no mutation gate (no runnable SUT, or a quick pass)
npx falsegreen-skill fix tests/test_discount.py --case C5 --line 9 --cheap

# machine-readable gate verdict (schema/fix-validation.json)
npx falsegreen-skill fix tests/test_discount.py --case C20 --line 22 --sut src/discount.py --json
```

| Flag | Description |
|---|---|
| `--case <code>` | Catalog code of the finding to fix (`C2b`, `C20`, `C21`, `C5`, `C7`) |
| `--line <n>` | Line of the finding in the test file (1-indexed) |
| `--sut <file>` | Production file the test protects. Required for a validated fix; without it the gate degrades to propose-only |
| `--sut-line <n>` | Line in the SUT to mutate (the behavioural line the finding names). Defaults to `--line` |
| `--cheap` | Validation tier: parse + preserve only, no mutation gate. Default tier is strong (parse + preserve + mutation) |

The gate runs three checks on a clean replica: the patch parses, it passes `pytest` against the
real code, and it **fails** on a line-scoped mutation of the SUT. A patch is accepted only when it
both passes on correct code and goes red on the mutant, which is what proves the new assertion
catches a bug instead of being a fresh tautology. Without `--sut` it degrades to propose-only and
says the fix is unvalidated. The honest limit: the gate proves the fix catches the targeted mutant,
not every possible bug.

#### `generate` mode (self-check) { #generate-mode }

`generate` is Mode B: it renders a language-neutral test spec into a real test, then runs Mode A on
the result so a false-green shape cannot pass the self-check undetected. Authoring now has a CLI
surface, not only the host-side skill.

```text
falsegreen-skill generate <spec-file> [--lang <language>] [options]
```

| Flag | Description | Default |
|---|---|---|
| `--lang <language>` | `python`, `typescript`, `javascript`, `tsx`, `jsx`, or `robot`. `tsx`/`jsx` cover the React side of the JS/TS family (same shared catalog `falsegreen-js` applies over `.js`/`.ts`/`.tsx`/`.jsx`). One language per run - the spec is the single source, re-run to render another stack | spec's first `languages` entry, else `python` |

```bash
# render the example spec to a Python test and self-check it
npx falsegreen-skill generate examples/authoring/apply-discount.spec.yaml --lang python

# same spec, TypeScript
npx falsegreen-skill generate examples/authoring/apply-discount.spec.yaml --lang typescript

# machine-readable: { language, test, self_check, self_check_passed, self_check_error }
npx falsegreen-skill generate my-spec.yaml --lang python --json
```

An offline oracle guard runs before any API call: `generate` refuses, exit 1, when the spec carries
no `oracle:` / `expected:` key (also when the language is unknown or the file is missing). A test
written from the code's current output only freezes the bug (a characterization test). The guard
anchors on the keys, so the words in a comment no longer pass; the oracle's *value* is still yours
to get right.

After generating, the CLI runs Mode A on the test and the exit code fails closed:

| Exit | State | Meaning |
|---|---|---|
| `0` | PASSED | self-check ran, no HIGH false-green |
| `1` | FAILED | a surviving false-green was confirmed (also: bad spec / API error via the offline guard) |
| `3` | UNVERIFIED | the self-check could not run; the test prints to stdout but is **not** accepted |

The honest limit: the self-check is a same-model static review, not an execution. It does not run
the test, confirm it compiles or imports, or verify the oracle value.

#### Exit codes { #cli-exit-codes }

| Code | Meaning |
|---|---|
| `0` | Analysis completed (findings may still exist; `analyze` is an analysis tool, not a gate) |
| `1` | Error: missing file, missing API key, bad flag, invalid JSON, schema mismatch, non-2xx API response |
| `2` | `--fail-on-high` was set and the JSON report contains at least one HIGH finding |

### Provider configuration { #provider-configuration }

The skill is not tied to Claude. The protocol is provider-agnostic; pick a backend with
`--provider` and set the matching API key in the environment.

| Variable | Used by |
|---|---|
| `ANTHROPIC_API_KEY` | `--provider anthropic` |
| `OPENAI_API_KEY` | `--provider openai` (and fallback for `openai-compatible`) |
| `GEMINI_API_KEY` | `--provider gemini` |
| `FALSEGREEN_API_KEY` | `--provider openai-compatible` (takes precedence over `OPENAI_API_KEY`) |

Default models: `anthropic` uses `claude-sonnet-4-6`, `openai` uses `gpt-4o`, `gemini` uses
`gemini-2.5-pro`. For deep case 18 analysis, pass `--model claude-opus-4-8` (Anthropic) or
`--model o3` (OpenAI); when using `o3`, `--temperature` is ignored automatically.

```bash
# Anthropic (default)
export ANTHROPIC_API_KEY=sk-ant-...
falsegreen-skill analyze tests/test_payment.py

# OpenAI
export OPENAI_API_KEY=sk-...
falsegreen-skill analyze tests/test_payment.py --provider openai

# Google Gemini
export GEMINI_API_KEY=...
falsegreen-skill analyze tests/test_payment.py --provider gemini
```

#### openai-compatible (base-url) { #openai-compatible }

Any OpenAI-compatible endpoint works through `--provider openai-compatible` with an explicit
`--base-url` and `--model`. Set `FALSEGREEN_API_KEY` to that provider's key.

```bash
# Groq (fast LLaMA)
export FALSEGREEN_API_KEY=gsk_...
falsegreen-skill analyze tests/test_payment.py \
  --provider openai-compatible \
  --base-url https://api.groq.com/openai/v1 \
  --model llama-3.3-70b-versatile

# Ollama (local)
export FALSEGREEN_API_KEY=ollama
falsegreen-skill analyze tests/test_payment.py \
  --provider openai-compatible \
  --base-url http://localhost:11434/v1 \
  --model qwen2.5-coder:32b
```

Reasoning models on openai-compatible hosts use the same shape, just a stronger model id:

```bash
# Nvidia NIM (DeepSeek R1 reasoning)
export FALSEGREEN_API_KEY=nvapi-...
falsegreen-skill analyze tests/test_orders.py \
  --provider openai-compatible \
  --base-url https://integrate.api.nvidia.com/v1 \
  --model deepseek-ai/deepseek-r1

# Fireworks (DeepSeek R1 reasoning)
export FALSEGREEN_API_KEY=fw_...
falsegreen-skill analyze tests/test_orders.py \
  --provider openai-compatible \
  --base-url https://api.fireworks.ai/inference/v1 \
  --model accounts/fireworks/models/deepseek-r1

# Groq (DeepSeek R1 distill, reasoning)
export FALSEGREEN_API_KEY=gsk_...
falsegreen-skill analyze tests/test_orders.py \
  --provider openai-compatible \
  --base-url https://api.groq.com/openai/v1 \
  --model deepseek-r1-distill-llama-70b
```

For the hardest [case 18](../catalog/semantic.md) findings (expected value contradicts the spec),
the maintained pattern is a two-pass finder/refuter call with a reasoning model, documented in the
project's `providers.md`.

### Per-host enable { #per-host-enable }

The same protocol is packaged for each host. Pick the path that matches your editor or agent.

**Claude Code (primary path).** Add the marketplace and install the plugin:

```text
/plugin marketplace add vinicq/falsegreen-skill
/plugin install falsegreen-skill@falsegreen
```

Then invoke `/falsegreen-skill:falsegreen-llm`, or attach a test file and ask for false-positive
analysis, the skill triggers on intent. For Python it applies the full pattern catalog directly;
optionally run the static [Python scanner](python.md) first and hand its output to the skill as
the structural pass.

**OpenAI Codex CLI.** No single-command install for this repo. Clone it and run `codex` for the
full protocol (the root `AGENTS.md` auto-loads, scoped to the clone), or copy `AGENTS.md` into
your own project or `~/.codex/AGENTS.md` for the compact routine protocol.

**Antigravity CLI (`agy`).** Google's successor to the discontinued Gemini CLI. Install it as a
plugin:

```bash
agy plugin install https://github.com/vinicq/falsegreen-skill
```

Or run `agy` inside the repo with no install: it discovers the workspace skill at
`.agents/skills/falsegreen-skill/SKILL.md` and parses `AGENTS.md` / `GEMINI.md` at the root as rule
files on startup. The skill is the `/falsegreen-skill` command; ask in natural language. If you
still have the old Gemini CLI extension, migrate it with `agy plugin import gemini`.

**Cursor.** Copy the rule template into `.cursor/rules/falsegreen-skill.mdc` (full template in the
project's `contexts/cursor.md`). Cursor loads it on a matching test file; ask Cursor to analyze the
file and the J1-J6 protocol runs.

**Plain LLM / API.** Use the SDK snippets in the project's
[providers.md](https://github.com/vinicq/falsegreen-skill/blob/master/providers.md): the system
prompt is `SKILL.md`, the user message is the test file, and structured JSON output follows
`schema/report.json`. The same base-url pattern as the CLI covers any OpenAI-compatible backend.

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
