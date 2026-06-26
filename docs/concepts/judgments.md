# Judgments (J1-J6)

A judgment is a question asked of a single test. Six questions decide whether a test really
protects anything. Each catalog code carries the judgment it answers, so a finding is never
just "something looks off": it names which guarantee the test fails to provide.

| Judgment | The question | A test fails it when |
|---|---|---|
| **J1** | Does the assertion run? | the check is missing, unreachable, swallowed, or skipped |
| **J2** | Is the expected value from an independent oracle? | the expected value is always-true, self-referential, or copied from the output |
| **J3** | Is the real unit exercised? | the test asserts a mock, a stub, or its own setup |
| **J4** | Is the assertion sufficient? | the check is too weak or too broad to fail on the real defect |
| **J5** | Is it free of coupling to internals? | the test reads private fields or implementation detail |
| **J6** | Does it pass in isolation? | the result depends on order, shared state, time, or randomness |

## How a judgment becomes a code

The judgment is the *why*; the code is the *what a tool can prove*. One judgment covers many
codes across languages:

- **J1** (the assertion does not run) covers the empty test (`C2`), the assertion after a
  `return` (`C20`), the swallowed `try/except` (`C3`), the commented-out check (`CC`), the
  skipped test (`C32`), and their JS and Robot equivalents.
- **J2** (the expected value is not independent) covers the always-true assertion (`C5`), the
  self-comparison (`C7`), the numeric tautology (`C44`), and the golden file copied from output
  (`C14`).
- **J3** (the real unit is not exercised) covers mocking the unit under test (case 10), the
  self-confirming literal (`C11a`), and patching core logic (`S12`).
- **J4** (the assertion is insufficient) covers the weak truthiness check (`C6`), the broad
  `pytest.raises(Exception)` (`C9`), and the coarse oracle the skill flags (`S4`).
- **J5** (coupling to internals) covers reading underscore-prefixed private fields and
  string/repr comparisons that bind to implementation detail.
- **J6** (does not pass in isolation) covers shared mutable state (`C24`), order dependence
  (case 15), uncontrolled time or randomness (`C16`), and flaky-retry decorators (`C35`).

## Why six and not one

A single "is this a good test?" verdict hides the reason and invites argument. Splitting it
into six independent questions makes each finding defensible: the tool points at exactly one
broken guarantee, shows the signal, and leaves the other five out of it. It also keeps false
positives down, because a pattern only fires when it clearly fails a specific judgment, not on
a vague sense of smell.

The semantic pass leans hardest on J2, J3, and J4, because those need reading the expected
value against the spec and the production code, which no parser can do. The static scanners own
the J1, J5, and J6 cases a parser can prove, plus the structural slice of J2 and J4.
