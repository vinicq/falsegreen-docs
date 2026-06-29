# Coverage by failure family

The [denominator page](denominator.md) names the universe the family is measured against (the
[Open Catalog of Test Smells](https://test-smell-catalog.readthedocs.io/), 517 documented smells)
and the axis in scope (false-green only). This page goes one level finer: per failure family
([F1-F8](taxonomy.md)), which codes the four scanners actually ship, with a link to the public
code that proves it.

## What this page counts, and what it does not

This is a **public-code** view, not an evaluation. It counts codes the ecosystem ships per family,
each linked to the catalog entry or the scanner that emits it. It does **not** report precision or
recall against the catalog, and it carries no dataset evidence. Those numbers are measured against
the false-green slice of the denominator and released with the study, not here. The honest reading
of the table below: "the family ships these codes for this failure mode," not "the family detects
N% of the literature."

A code can appear under more than one scanner when the same id covers the same mechanism in a
different language (C5 is the always-true assertion in Python, JS/TS, and Robot). Counting a family
by distinct ids, not by per-scanner rows, avoids double counting.

## The four scanners

| Scanner | Language | Public catalog (the code list) |
|---|---|---|
| [falsegreen](https://github.com/vinicq/falsegreen) | Python / pytest | [README catalog](https://github.com/vinicq/falsegreen#what-it-detects) · [`scanner.py`](https://github.com/vinicq/falsegreen/blob/main/src/falsegreen/scanner.py) |
| [falsegreen-js](https://github.com/vinicq/falsegreen-js) | JS / TS | [README catalog](https://github.com/vinicq/falsegreen-js#readme) |
| [robotframework-falsegreen](https://github.com/vinicq/robotframework-falsegreen) | Robot Framework | [README catalog](https://github.com/vinicq/robotframework-falsegreen#readme) |
| [falsegreen-skill](https://github.com/vinicq/falsegreen-skill) | semantic (LLM) | [`reference.md`](https://github.com/vinicq/falsegreen-skill/blob/master/reference.md) (the superset) |

The skill is the superset: every structural code the three static scanners emit appears in its
[`reference.md`](https://github.com/vinicq/falsegreen-skill/blob/master/reference.md), plus the
semantic-only codes (cases 10, 11, 12, 15, 18) that no parser can reach.

## Codes per family

The taxonomy ([F1-F8](taxonomy.md)) is the conceptual axis: how a test goes green without
protecting anything. The codes below are the public ids the ecosystem ships for each mode. Static
scanners cover F1-F3, F5, and the static proxies of F6; the skill adds F4 and F7; F8 is the
diagnostic group, off by default.

| Family | Failure mode | Codes the ecosystem ships | Layer |
|---|---|---|---|
| **F1** | Checks nothing (no oracle) | `C2`, `C2b`, `C2c`, `C27`, `C39`, `C50`, `C51`, `JS2`, `JS6`, `JS13`, `R2`, `R4`, `R7`, semantic cases 10/11 | static + skill |
| **F2** | The check exists but never runs | `C1`, `C3`, `C20`, `C21`, `C22`, `C43`, `CC`, `JS5`, `JS7`, `JS9`, `JS11`, `JS25`, `JS26`, `JS29`, `JS31`, `R8`, `R8b` | static |
| **F3** | The check is trivial (always passes) | `C5`, `C6`, `C6c`, `C7`, `C8`, `C8b`, `C11a`, `C18`, `C34`, `C42`, `C44`, `C52`, `JS15`, `JS21`, `JS30`, `R1`, `R6` | static |
| **F4** | Checks the wrong thing | `C9`, `C9b`, `C19`, `C28`, `C49`, `C55`, `JS8`, `JS24`, `JS27`; semantic case 18, parts of `C6` / `C33` / the snapshot codes | static (partial) + skill |
| **F5** | Drops out of the count (skip / not collected) | `C4`, `C4b`, `C25`, `C32`, `C38`, `C45`, `JS1`, `JS4`, `JS22`, `JS23`, `R3`, `R5`; project layer: `PL1`, `PL2`, `PL7`, `PL8`, `PL9`, `PL10` | static + project layer |
| **F6** | Passes or fails by luck (non-determinism) | `C16`, `C23`, `C24`, `C29`, `C35` (static proxies) | static (proxy) + runtime |
| **F7** | Circular or semantic oracle | semantic cases 10, 11, 12, 15; `C14` (the codable corner) | skill + mutation testing |
| **F8** | Hygiene / readability (not false-green) | `D1`, `D3`, `D4`, `D5`, `D6`, `D7`, `D8`, `M2` (opt-in diagnostics) | diagnostic / linter |

The exact, current code list per scanner lives in each repository's README catalog and in the
skill's [`reference.md`](https://github.com/vinicq/falsegreen-skill/blob/master/reference.md). This
table groups those published codes by failure mode; it does not invent new ones. Where a code maps
to more than one family (a code can be both "never runs" and "weak"), it is listed under the family
that names its primary mechanism.

## What is and is not counted per family

- **F1, F2, F3** are fully static and saturated: a per-file parser proves them with no false
  negatives inside its rules. The scanner READMEs list every id.
- **F4** is counted only for the slice a parser can reach (a string-format comparison, a discarded
  metric). The contradicts-the-spec core is semantic and lives in the skill (case 18); it is not a
  static count.
- **F5** has two slices: the per-file slice (a test not collected, a non-strict xfail) counted in
  the scanner codes, and the project slice (`PL1`, `PL2`, `PL7`, `PL8`, `PL9`, `PL10`, read by `--config-audit`) counted
  separately. The runtime slice (a collection error reported as "0 tests") is documented, not a
  code.
- **F6** is counted only as static proxies (`C16` for uncontrolled time/randomness, `C23` for a
  hard-coded path). Whether a test is flaky in practice needs runtime and is out of band, so it is
  not counted here.
- **F7** is the semantic family. Only `C14` (a snapshot generated from the code's own output) is a
  static code; the rest (mocking the unit under test, re-implementing the formula, borrowed state)
  are skill cases and are confirmed with mutation testing, which the skill never runs itself. They
  are listed, not counted as static coverage.
- **F8** is not false-green. The diagnostic codes are off by default, and dedicated linters (ruff,
  ESLint, Robocop) cover the same ground. They are surfaced on request, not promised as detection.

## Why this is the honest framing

A coverage claim is only meaningful against a named denominator. The percentage of the catalog the
family detects is reported against the [false-green slice](denominator.md), not the whole 517, and
that number ships with the study. This page stops at what the public code proves: which codes exist
per family, where the code is, and which layer owns each mode. The boundary in the
[scope page](scope.md) and the cross-walk in the [denominator page](denominator.md) say what stays
out and why.
