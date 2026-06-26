# falsegreen-skill (semantic LLM pass)

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

## Install

```bash
npm install -g falsegreen-skill
```

## Modes

- **Detect** - read a suite and report findings (J1-J6, level, evidence, fix hint).
- **Author** - generate tests that are not false-green by construction, one spec per pyramid
  level.
- **AI-fix gate (F7)** - propose a strengthened test and validate it with a bidirectional
  mutation gate (pass on clean code, fail on the reintroduced bug). The skill proposes; the host
  runs mutation testing (mutmut, cosmic-ray, Stryker).

## Scope

The broadest tool of the family, still false-green only. See
[scope and honesty](../concepts/scope.md) and [coverage vs the literature](../concepts/denominator.md).
