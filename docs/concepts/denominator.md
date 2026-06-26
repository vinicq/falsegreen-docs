# Coverage versus the literature

The family measures itself against the [Open Catalog of Test Smells](https://test-smell-catalog.readthedocs.io/)
(UFAL / easy-software): 517 documented smells across 1621 references and 166 sources. Naming the
denominator keeps the claims honest. It says what universe the tools were checked against, and,
just as important, what stays out of scope on purpose.

## One axis: false-green

Every tool in the family detects one thing: a test that passes green without protecting
anything (the [F1-F8 taxonomy](taxonomy.md)). The overwhelming majority of the catalog is a
different concern. Mixing those in would raise the false-positive rate, and a tool that cries
wolf gets turned off.

## What stays out, and why

| Category | Why it is out | Examples (catalog id) |
|---|---|---|
| **Brittleness / false-red** | breaks *without* a real bug; the opposite axis | Sensitive Equality, Brittle Assertion, Fragile Fixture, Interface/Behavior Sensitivity, Overspecified Software |
| **Hygiene / maintainability** | the test still protects; it is just hard to read (linter territory: ruff, ESLint, Robocop) | Assertion Roulette, Magic Number Test, Long Test, Verbose Test, Overcommented Test, Missing Assertion Message |
| **Slow / performance** | execution time, invisible in the file | Slow Test, Slow Component Usage |
| **Design** | architecture, not the green of the test | Constructor Initialization, Refused Bequest, Hard-To-Test Code |
| **Naming / docs** | readability; a linter owns it | Anonymous Test, Bad Naming, Absence Of Why, Bad Comment Rate |
| **Duplication** | maintainability | Test Code Duplication, Duplicated Code, Duplicate Assert |
| **Runtime / culture** | needs the suite to run, or is a team practice | Test Run War, Erratic Test, Manual Intervention, Frequent Debugging |

The catalog uses its own id scheme (`A16`, `S06`, `C30`), which is not the same as the family's
codes (`C1`, `C23`). The ids above are the catalog's.

## The boundary is deliberate, not accidental

Several smells look out of scope but have a false-green *proxy* the family does catch. The tools
take the statically provable, low-false-positive form of each boundary and leave the rest:

| Catalog smell | The proxy the family catches |
|---|---|
| Nondeterministic / flaky test | `C16` (uncontrolled time, randomness, or sleep) |
| Resource Optimism | `C23` (hard-coded path or IP URL) |
| Interacting / order-dependent tests | `C24` / `C15` / `S13` (shared state) |
| Conditional Test Logic | `C1` / `C21` (an assertion that may never run) |
| No Assertions / Empty / Assertionless Test | `C2` / `C2b` |
| Rotten Green Test | the whole product: the false-green axis |

A few hygiene smells are surfaced as **opt-in diagnostics** (off by default, not false-green):
Assertion Roulette as `D1`, Magic Number as `D8`, Duplicate Assert as `D3`, Long Test as `M2`.
Where a dedicated linter covers a smell better, the scanners defer to it.

## Why this matters for research

This page is the threats-to-validity statement in public form. Precision and recall are reported
against the false-green slice of the catalog, not the whole 517, and the boundary table above
shows exactly where the line sits. The full cross-walk (every catalog smell mapped to in-scope or
out, with the reason) lives in the private research hub; only the denominator and the boundary are
public, never the raw adjudication.
