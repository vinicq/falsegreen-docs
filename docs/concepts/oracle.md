# The oracle hierarchy

An oracle is the source of the expected value: what the test compares the code against. A test
is only as honest as its oracle. If the expected value comes from the code itself, the test
confirms the code matches itself, which is true even when the code is wrong.

The expected value must come from a source independent of the code, in this order:

1. **Explicit spec or requirement** - a spec document, ticket, or RFC. The strongest oracle:
   it predates the code and does not change when the code changes.
2. **Documented contract** - a docstring, type annotations, API docs. Weaker than a spec but
   still independent of the implementation.
3. **Independent human judgment** - the tester derives the expected value from the requirement
   by hand, without reading the current output.
4. **The current code** - the lowest priority. This is where bugs hide.

## Why the order matters

Promoting the current code to the top of this hierarchy is how a bug gets frozen as "correct".
The pattern looks reasonable: run the function, capture what it returns, paste that as the
expected value. From then on the test passes as long as the code keeps doing what it does now,
including the bug.

The catalog has several codes for exactly this inversion:

- **`C14`** - the golden file written from the actual output on first run.
- **`C12`** - the test re-implements the production formula, so both sides agree on the same
  wrong answer.
- **`C11` / `C11a`** - the test asserts the value it just fed in.
- **`S7`** - the semantic version: the expected value was lifted from a run of the current
  code (a pasted dict, a captured response).
- **Case 18 / `S3`** - the expected value contradicts the spec, so the test froze a bug as the
  correct behavior. These need an independent oracle cited before the finding is reported.

## The reporting rule

For any finding that depends on the expected value being wrong, the skill does not report
without citing an oracle. "This number looks wrong" is not enough; the finding must point at
the spec, the docstring, or the derivation that says what the number should be. A false
positive here is expensive, so the bar is an explicit, independent source.

This is the principle behind the whole family: the static scanners catch the structural
inversions a parser can prove (`C14`, `C12`, `C11a`), and the [semantic pass](../scanners/skill.md)
catches the ones that need reading the expected value against the spec.
