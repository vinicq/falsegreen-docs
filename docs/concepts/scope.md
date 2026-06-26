# Scope and honesty

The family is built on one definition: a false-green test reports success while the code it
covers may be broken. Everything inside that line is in scope. Everything outside it is named
here, with the reason it stays out, because a tool that overreaches loses the trust that makes
it useful.

## What the static scanners prove

A parser proves what is structurally true in a single file, without running anything:

- the assertion is missing, unreachable, or swallowed (F1, F2);
- the assertion is always true by construction (F3);
- the test drops out of the run (F5, the per-file slice);
- a static proxy for non-determinism: a hard-coded path, an unfrozen clock (F6, partial).

This is fast, deterministic, and has no false negatives within its rules. It also has a ceiling:
roughly 90% of the false-green *mechanisms* that a syntax can express. Pushing for more codes
here trades signal for noise.

## What the skill adds

The [semantic pass](../scanners/skill.md) reads the test against its intent, the spec, and the
production code. It catches what no parser sees (F4, F7):

- the expected value contradicts the spec;
- the test mocks the unit it claims to test;
- the oracle is too coarse to fail on the real defect;
- the assertion checks an irrelevant property.

The skill is the superset: it carries every structural code plus the semantic ones. A false
positive here is still worse than a miss, so the semantic findings show their reasoning and
cite an oracle before they report.

## What needs runtime, and is not promised statically

Some false-green modes only appear when the suite runs:

- `python -O` stripping `assert`, a collection error reported as "0 tests passed", a CI step
  that runs a subset and reports green. These are the runtime slice of F5.
- Whether a strengthened test actually fails on a specific bug. That is the
  [AI-fix gate](../scanners/skill.md), proved with mutation testing (mutmut, cosmic-ray,
  Stryker), which the skill never runs itself.

These are documented, not claimed. A tool that promised them statically would be lying.

## What is deliberately out

- **False-red, brittleness, flakiness.** Tests that break *without* a real bug are the
  opposite axis. Mixing them with false-green produces noise and contradicts the product
  definition. `C8` (exact float equality) and `C16` (sources of non-determinism) are the two
  static proxies that sit on the false-green side of the line; the rest stays out.
- **Pure hygiene.** Dead code, unused arguments, long methods, missing docs. These are F8 and
  belong to dedicated linters (ruff, ESLint, Robocop). The diagnostic group surfaces a few on
  request, off by default.
- **Coverage, performance, culture.** Slow tests, test-run war, human quality gates. Invisible
  in the file; a matter of process or runtime, not a false-green pattern.

The honest summary on every scanner README says the same thing: static proves the structural
slice, the skill reads the semantic slice, runtime is out of band. Each tool states what it
does not do, so the green it gives you means something.

For the full picture of what stays out and why, measured against the published test-smell
literature, see [coverage versus the literature](denominator.md).
