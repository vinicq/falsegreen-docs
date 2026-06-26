# falsegreen

**One problem, one tool: the false positive.** falsegreen finds tests that pass green
without protecting anything. A test goes green because it asserts nothing, asserts something
always true, never reaches the assertion, or checks the wrong thing. The code underneath can
be broken and the suite still reports success.

A test that tells you a broken program is safe is worse than no test at all. It buys false
confidence, and false confidence ships bugs. AI coding assistants produce these tests at
scale, which is why the problem is worth a dedicated tool.

This site is the shared reference for the whole family: every detection code, the exact signal
each one keys on, and a BAD example next to the CLEAN look-alike so you can see the difference.

[Browse the catalog](catalog/index.md){ .md-button .md-button--primary }
[How the codes are classified](concepts/taxonomy.md){ .md-button }

## The family

Four tools, one catalog. The three static scanners prove what a parser can see, each in its
own language. The skill is the semantic pass that reads intent, and a superset of the three.

| Tool | Language | Technique |
|---|---|---|
| [falsegreen](scanners/python.md) | Python / pytest | AST scan, zero dependencies |
| [falsegreen-js](scanners/javascript.md) | JavaScript / TypeScript | TypeScript compiler API |
| [robotframework-falsegreen](scanners/robot.md) | Robot Framework | `robot.api` model |
| [falsegreen-skill](scanners/skill.md) | Python, TS, JS, Robot | semantic LLM pass (J1-J6) |

Codes share an id where the concept matches across languages: `C5` is the always-true
assertion in Python, JavaScript, and Robot alike. Language-specific patterns get their own
series (`JS*` for JavaScript, `R*` for Robot, `S*` for the semantic pass).

## Three layers of false-green

Not every false-green test is visible to a parser. The catalog is organized around where the
problem lives, because that decides which tool can catch it.

1. **Static, per file.** The assertion is empty, always true, unreachable, or swallowed. A
   parser proves it without running anything. This is what the three scanners do.
2. **Project and CI.** The file is clean but the suite still lies: a config that lets an empty
   run pass, a coverage gate that is never enforced, warnings that never become errors. This
   is the `--config-audit` mode.
3. **Semantic and runtime.** The oracle is circular, the expected value contradicts the spec,
   the test mocks the very unit it claims to test. No AST sees intent. The skill reads it; a
   live gate confirms it with mutation testing.

See [scope and honesty](concepts/scope.md) for what each layer can and cannot prove.

## How to read a code

Every entry in the catalog carries the same fields:

- **What it detects** - the failure in plain terms.
- **Signal** - the concrete syntactic trigger the tool keys on, and when it deliberately
  holds back to avoid a false positive.
- **BAD / CLEAN** - the flagged pattern next to a look-alike that is correct, so the boundary
  is visible.
- **Judgment** (J1-J6) and **family** (F1-F8) - where the code sits in the
  [taxonomy](concepts/taxonomy.md).
- **Confidence** - HIGH blocks, LOW warns, OFF is diagnostic-only.

The guiding rule across the whole family: **a false positive is worse than a miss.** A tool
that cries wolf gets turned off, and a tool that is off protects nothing. Every code is tuned
to fire on a real false-green and stay quiet on the look-alike that resembles it.
