# Catalog overview

Every detection code in the family, in one place. Each language page opens with a compact index
table (code, confidence, judgment, one-liner) that links straight to that code's entry below.
The entries run one code at a time, grouped by family, each naming the exact signal it keys on
with a BAD example next to the CLEAN look-alike, so the boundary is visible.

Use the search box (top right) to jump to any code by id (`C5`, `JS21`, `R2`) or by keyword
(`mock`, `skip`, `always-true`). Every code has its own anchor, so reports and tools can deep
link straight to it.

## The pages

- [Python](python.md) - the `C*` structural codes, the `PL*` config-audit layer, and the
  diagnostic group, as implemented by the [falsegreen](../scanners/python.md) scanner.
- [JavaScript / TypeScript](javascript-typescript.md) - the shared `C*` codes, the `JS*`
  ecosystem-specific ones, the `PL*` config-audit layer, and the diagnostic group, from
  [falsegreen-js](../scanners/javascript.md).
- [Robot Framework](robot.md) - the shared `C*` codes, the `R*` ones, the `PL*` layer, and the
  diagnostic group, from [robotframework-falsegreen](../scanners/robot.md).
- [Semantic (LLM)](semantic.md) - the canonical cases plus the `S1-S21` series that only the
  [skill](../scanners/skill.md) can catch.
- [Gherkin and Tavern](gherkin-tavern.md) - the semantic, text-based pass over `.feature` and
  `*.tavern.yaml` files.

Three groupings recur across the structural pages: the per-file codes (the `C*`, `JS*`, `R*`
series), the **project layer** (`PL*`), emitted by the config-audit pass when the suite goes
green by runner configuration rather than a smell in any one file, and the **diagnostic group**
(`D*`, `M2`), opt-in and OFF by default, which flags hygiene that is not false-green.

## How a code is shared across languages

A code keeps its id where the concept is the same. `C5` is the always-true assertion whether it
is written `assert True` in Python, `expect(true).toBe(true)` in JavaScript, or
`Should Be True    ${TRUE}` in Robot. The signal differs by language; the failure mode does
not. Language-specific patterns get their own series: `JS*` for JavaScript idioms, `R*` for
Robot constructs, `S*` for the semantic pass.

## The legend

Each entry is tagged with three axes.

**Confidence** decides what the tool does with the finding:

| Confidence | Meaning | Effect |
|---|---|---|
| **HIGH** | a definite false-green | blocks (non-zero exit) |
| **LOW** | a likely smell, operator confirms | warns |
| **OFF** | diagnostic only | shown on request, never blocks |

**Judgment** (J1-J6) names the broken guarantee. See [judgments](../concepts/judgments.md).

| | |
|---|---|
| **J1** | the assertion does not run |
| **J2** | the expected value is not from an independent oracle |
| **J3** | the real unit is not exercised |
| **J4** | the assertion is insufficient |
| **J5** | the test couples to internals |
| **J6** | the test does not pass in isolation |

**Family** (F1-F8) names the failure mode. See [taxonomy](../concepts/taxonomy.md).

| | |
|---|---|
| **F1** | checks nothing | **F5** | drops out of the count |
| **F2** | the check never runs | **F6** | passes or fails by luck |
| **F3** | the check is always true | **F7** | circular or semantic oracle |
| **F4** | checks the wrong thing | **F8** | hygiene, not false-green |

## A note on look-alikes

For most codes the catalog also shows what *not* to flag: the pattern that resembles the smell
but is correct. A self-comparison that also tests `__eq__`, a truthiness check at the HTTP
layer, a typed mock wrapper. These exemptions are why the false-positive rate stays low, and
they are listed on each page.
