# JavaScript / TypeScript catalog

The codes implemented by [falsegreen-js](../scanners/javascript.md): a static scan via the
TypeScript compiler API, runner-agnostic across Jest, Vitest, Mocha+Chai, Jasmine, AVA,
node:test, Cypress, Playwright, and Testing Library. Covers `.js`, `.jsx`, `.ts`, `.tsx`,
`.mjs`, `.cjs`, `.mts`, `.cts`.

Codes share an id with [Python](python.md) where the concept matches; `JS*` codes are
ecosystem-specific. Confidence: **HIGH** blocks, **LOW** warns. Judgments are
[J1-J6](../concepts/judgments.md).

Every emitted code has its own entry below. The index links straight to each one.

## Index

| Code | Conf | J | One-liner |
|---|---|---|---|
| [C2](#c2) | HIGH | J1 | empty test body |
| [C2b](#c2b) | LOW | J1 | calls the unit but never asserts |
| [C5](#c5) | HIGH | J2 | always-true (`expect(true).toBe(true)`) |
| [C6](#c6) | LOW | J4 | weak check (`toBeTruthy`, `.length > 0`) |
| [C7](#c7) | HIGH | J2 | self-compare (`expect(x).toBe(x)`) |
| [C8](#c8) | LOW | J4 | exact equality on a float |
| [C8b](#c8b) | LOW | J4 | `toBeCloseTo` with no precision argument |
| [C9](#c9) | LOW | J4 | `toThrow()` with no error type or message |
| [C11a](#c11a) | LOW | J2 | self-confirming literal bound from the call under test |
| [C16](#c16) | LOW | J1 | depends on time, randomness, or a fixed timer |
| [C18](#c18) | LOW | J2 | stringified equality (`String(x)`/`JSON.stringify`) |
| [C20](#c20) | HIGH | J1 | dead assertion after `return`/`throw` |
| [C21](#c21) | LOW | J1 | no assertion runs unconditionally |
| [C23](#c23) | LOW | J6 | reads a real file at a literal path / hard-coded URL |
| [C37](#c37) | LOW | J4 | duplicate `it.each`/`test.each` case |
| [C44](#c44) | HIGH | J2 | numeric tautology on a length (`.length >= 0`) |
| [C48](#c48) | LOW | J1 | dark patch: flips a test-mode flag then asserts |
| [CC](#cc) | LOW | J1 | commented-out assertion |
| [JS1](#js1) | HIGH | J1 | focused test skips the rest of the suite |
| [JS2](#js2) | HIGH | J1 | `expect` with no matcher |
| [JS3](#js3) | LOW | J2 | snapshot is the only assertion |
| [JS4](#js4) | LOW | J1 | skipped test (`it.skip`/`xit`/`it.todo`) |
| [JS5](#js5) | LOW | J1 | async query/event not awaited |
| [JS6](#js6) | HIGH | J1 | empty `describe`/suite |
| [JS7](#js7) | LOW | J1 | assertion in a non-awaited `setTimeout`/`then` |
| [JS8](#js8) | LOW | J3 | mocks the unit under test and asserts it directly |
| [JS9](#js9) | HIGH | J1 | assertion in a dead literal branch |
| [JS11](#js11) | LOW | J1 | try/catch swallows the assertion |
| [JS13](#js13) | LOW | J4 | query used as a loose statement, never asserted |
| [JS15](#js15) | LOW | J4 | comparison wrapped in a boolean |
| [JS17](#js17) | LOW | J1 | commented-out test block |
| [JS18](#js18) | LOW | J1 | `done` callback instead of async/await |
| [JS21](#js21) | HIGH | J1 | matcher referenced but never called |
| [JS22](#js22) | HIGH | J1 | empty `it.each`/`test.each` table |
| [JS23](#js23) | HIGH | J1 | `expect.assertions(N)` the body cannot satisfy |
| [JS24](#js24) | LOW | J4 | Cypress query with no assertion |
| [JS25](#js25) | HIGH | J1 | the only assertion is inside an array-iterator callback |
| [JS26](#js26) | LOW | J1 | fake timers installed but never advanced |
| [JS27](#js27) | LOW | J3 | `toHaveBeenCalled*` is the sole oracle on a local double |
| [JS29](#js29) | LOW | J6 | `resolves`/`rejects` chain is a bare statement |
| [JS30](#js30) | HIGH | J2 | literal-vs-literal assertion |
| [JS31](#js31) | LOW | J1 | try/catch swallows a SUT throw, no assertion on the error |
| [D1](#diagnostic-codes-opt-in-off-by-default) | OFF | J4 | assertion roulette |
| [D3](#diagnostic-codes-opt-in-off-by-default) | OFF | J4 | duplicate assert |
| [D4](#diagnostic-codes-opt-in-off-by-default) | OFF | J4 | untitled `it.each` cases |
| [D6](#diagnostic-codes-opt-in-off-by-default) | OFF | J4 | `console.*` in a test body |
| [D7](#diagnostic-codes-opt-in-off-by-default) | OFF | J4 | anonymous test |
| [D8](#diagnostic-codes-opt-in-off-by-default) | OFF | J4 | magic number in an assertion |
| [M2](#diagnostic-codes-opt-in-off-by-default) | OFF | J5 | over-long test body |
| [PL7](#pl7) | LOW | J5 | no coverage gate |
| [PL8](#pl8) | LOW | J5 | `bail` stops the run early |
| [PL10](#pl10) | LOW | J1 | `passWithNoTests` lets an empty suite pass |

A code keeps its id across languages where the smell is the same; the signal below is the
JavaScript form. Two shared ids differ in meaning per language, see the [Python page](python.md):
**C31** does not exist here; **C44** is the numeric-tautology length check here and in Python,
widened in [Robot](robot.md#c44) to any vacuous library assertion.

## Shared codes (same concept as Python)

These mirror the [Python catalog](python.md) entries; the signal is the JavaScript form.

### C2 - empty test body { #c2 }
`J1` · HIGH · F1

A test with no matcher, no `expect`, no assertion of any kind. Always green.

### C2b - calls the unit but never asserts { #c2b }
`J1` · LOW · F1

The test calls production code but no matcher follows. The check is simply missing.

### C5 - always-true assertion { #c5 }
`J2` · HIGH · F3

`expect(true).toBe(true)`, `assert(1)`: structurally guaranteed to pass, so it protects nothing.

### C6 - weak check { #c6 }
`J4` · LOW · F4

`toBeTruthy()` / `toBeDefined()` or a `.length > 0` check: only that something came back, not the
value.

### C7 - self-compare { #c7 }
`J2` · HIGH · F3

`expect(x).toBe(x)`: both sides are the same expression, always equal.

### C8 - exact equality on a float { #c8 }
`J4` · LOW · F4

`expect(compute()).toBe(3.14159)`: floating-point arithmetic makes exact equality unreliable. Use
`toBeCloseTo`.

### C8b - `toBeCloseTo` with no precision argument { #c8b }
`J4` · LOW · F4

`toBeCloseTo(x)` defaults to 2-digit tolerance, which may be too loose for the value under test.
Pass an explicit precision.

### C9 - `toThrow()` with no error type or message { #c9 }
`J4` · LOW · F4

`expect(fn).toThrow()` with no argument accepts any error, including one from a typo inside the
test.

### C11a - self-confirming literal { #c11a }
`J2` · LOW · F3

The expected value is bound from the same call under test: `const e = foo(); expect(foo()).toBe(e)`.
The oracle confirms itself. Bind the expected value independently.

### C16 - depends on time, randomness, or a fixed timer { #c16 }
`J1` · LOW · F6

`Date.now` / `new Date()` / `Math.random` / `crypto.randomUUID` / `getRandomValues`, or a fixed
timer. Freeze time and seed randomness.

### C18 - stringified equality { #c18 }
`J2` · LOW · F4

Compares `String(x)` / `JSON.stringify(x)` / a template literal to a string literal. The format is
an implementation detail; assert the value.

### C20 - dead assertion after a terminator { #c20 }
`J1` · HIGH · F2

An assertion after `return` / `throw` / `process.exit` / an exhaustive `switch`. Structured
block-level reachability (cfg.ts) proves it never runs.

### C21 - no assertion runs unconditionally { #c21 }
`J1` · LOW · F2

Every assertion is inside a branch; none sits on the test's guaranteed spine. The same cfg.ts
reachability decides this, and a dead assertion does not mask it.

### C23 - reads a real file at a literal path or hard-coded URL { #c23 }
`J6` · LOW · F6

`readFileSync("/etc/...")` or a hard-coded URL: mystery guest. Use a fixture or temp file.

### C37 - duplicate `it.each` / `test.each` case { #c37 }
`J4` · LOW · F8

The same argument row appears twice in the table; the duplicate runs the same scenario and adds no
coverage.

### C44 - numeric tautology on a length { #c44 }
`J2` · HIGH · F3

`expect(x.length).toBeGreaterThanOrEqual(0)`: a length is never negative, so the comparison is
always true.

### C48 - dark patch: flips a test-mode flag then asserts { #c48 }
`J1` · LOW · F2

The test flips a test-mode flag (`process.env.NODE_ENV = "test"`, `process.env.TESTING = "1"`,
`settings.TESTING = true`) then asserts, so it exercises the product's test-only branch instead of
real behaviour. Parity with Python `C48`.

### CC - commented-out assertion { #cc }
`J1` · LOW · F2

A line in the body is a commented-out `expect(...)`: the check was switched off and left.

## JavaScript-specific codes

### JS1 - focused test skips the rest of the suite { #js1 }
`J1` · HIGH · F5

`it.only` / `fit` / `describe.only` silently excludes every other test in the file from the run.

=== "BAD"
    ```javascript
    it.only('adds', () => { expect(add(2, 3)).toBe(5); });
    it('subtracts', () => { /* never runs while .only is present */ });
    ```
=== "CLEAN"
    ```javascript
    it('adds', () => { expect(add(2, 3)).toBe(5); });
    it('subtracts', () => { expect(sub(3, 2)).toBe(1); });
    ```

### JS2 - expect with no matcher { #js2 }
`J1` · HIGH · F1

`expect(value)` with no matcher call. Nothing is asserted; the line is a no-op.

=== "BAD"
    ```javascript
    test('result', () => { expect(getResult()); });   // JS2 - no matcher
    ```
=== "CLEAN"
    ```javascript
    test('result', () => { expect(getResult()).toBe(42); });
    ```

### JS3 - snapshot is the only assertion { #js3 }
`J4` · LOW · F3

A test whose only check is `toMatchSnapshot()` / `toMatchInlineSnapshot()`. The baseline is
generated from the output, so it detects change, not correctness.

### JS4 - skipped test { #js4 }
`J1` · LOW · F5

`it.skip` / `xit` / `it.todo` left in place. Silently excluded from the run.

### JS5 - async query or event not awaited { #js5 }
`J1` · LOW · F2

A `promise`- or `value-only`-kind call (`findBy*`, `waitFor`, `userEvent`, a floating
`expect().resolves`) is dropped as a bare statement, so the following assertion reads a stale
moment. Detection routes through the oracle registry.

=== "BAD"
    ```javascript
    it('shows the row', () => {
      screen.findByText('Ada');                 // promise dropped
      expect(screen.getByRole('row')).toBeInTheDocument();
    });
    ```
=== "CLEAN"
    ```javascript
    it('shows the row', async () => {
      await screen.findByText('Ada');
      expect(screen.getByRole('row')).toBeInTheDocument();
    });
    ```

### JS6 - empty describe / suite { #js6 }
`J1` · HIGH · F1

A `describe`/`suite` block with no test inside. It contributes nothing and can read as coverage.

### JS7 - assertion in a non-awaited setTimeout / then callback { #js7 }
`J1` · LOW · F2

The assertion lives in a `setTimeout`/`setInterval` callback (timer arm) or a floating
`.then`/`.catch`/`.finally` (promise arm), so it may run after the test reported green.

=== "BAD"
    ```javascript
    it('fires later', () => {
      setTimeout(() => { expect(handler).toHaveBeenCalled(); }, 100);  // runs after the test ends
    });
    ```
=== "CLEAN"
    ```javascript
    it('fires later', async () => {
      jest.useFakeTimers();
      schedule();
      jest.runAllTimers();
      expect(handler).toHaveBeenCalled();
    });
    ```

### JS8 - mocks the unit under test and asserts it directly { #js8 }
`J3` · LOW · F4

The test mocks the very function it claims to test, then asserts the mock's value. It tests the
mock configuration, not the code. (The semantic, deeper form is case 10 / `S12`.)

### JS9 - assertion in a dead literal branch { #js9 }
`J1` · HIGH · F2

`if (false) { expect(...) }` or the `else` of `if (true)`. The assertion is unreachable.

### JS11 - try/catch swallows the assertion { #js11 }
`J1` · LOW · F2

The assertion sits in a `try` whose `catch` absorbs the thrown error (logs or ignores), so a
failure never propagates.

=== "BAD"
    ```javascript
    it('throws', () => {
      try { callUnit(); expect(true).toBe(false); } catch (e) { console.log(e); }
    });
    ```
=== "CLEAN"
    ```javascript
    it('throws', () => { expect(() => callUnit()).toThrow(RangeError); });
    ```

### JS13 - query used as a loose statement, never asserted { #js13 }
`J4` · LOW · F1

`getBy*` / `queryBy*` called as a bare statement. The query runs but nothing is checked on it.

### JS15 - comparison wrapped in a boolean { #js15 }
`J4` · LOW · F4

`expect(a === b).toBe(true)`. The boolean collapses the diff: on failure the message is
`false !== true`, with no values. Assert the values directly.

=== "BAD"
    ```javascript
    expect(user.id === 1).toBe(true);   // JS15
    ```
=== "CLEAN"
    ```javascript
    expect(user.id).toBe(1);
    ```

### JS17 - commented-out test block { #js17 }
`J1` · LOW · F2

`// it('...', ...)` left in the file. The test does not run.

### JS18 - done callback instead of async/await { #js18 }
`J1` · LOW · F2

A `done`-callback test where `done()` precedes or sits at the same level as the assertion, so the
runner completes before the assertion throws.

### JS21 - matcher referenced but never called { #js21 }
`J1` · HIGH · F2

`expect(x).toBe` with no `()`. The matcher is a property access; the check never runs. Sibling of
JS2.

=== "BAD"
    ```javascript
    expect(result).toBe;          // JS21 - missing (), does nothing
    ```
=== "CLEAN"
    ```javascript
    expect(result).toBe(42);
    ```

### JS22 - empty it.each / test.each table { #js22 }
`J1` · HIGH · F5

`it.each([])(...)`. Zero cases are generated, so the test never runs. Parallel of `C45`.

### JS23 - expect.assertions(N) the body cannot satisfy { #js23 }
`J1` · HIGH · F5

`expect.assertions(N)` with a numeric `N` higher than the unconditional, reachable, non-nested
`expect()` calls that can run. The guard the author wrote to defend against a silently-dropped
async assertion is dead on arrival. Fires only on a provable literal-`N` shortfall: an `expect`
in a loop, a branch, a `.then`/callback, or a called helper makes the count indeterminate and
suppresses the finding. `expect.hasAssertions()` carries no count and is skipped.

### JS24 - Cypress query with no assertion { #js24 }
`J4` · LOW · F4

A Cypress query chain (`cy.get`/`cy.find`/`cy.contains`) used as a statement with no terminating
`.should`/`.and` and no `expect` inside a `.then` callback. The query produces a subject that is
never asserted, the cy.* analogue of JS13. Action commands (`click`/`type`/`visit`/...) do work
rather than query, so a chain ending in one stays clean, as does one ending in `.should`/`.and`.

### JS25 - the only assertion is inside an array-iterator callback { #js25 }
`J1` · HIGH · F2

The sole `expect` sits inside a `forEach` / `map` / `filter` / `some` / `every` / `flatMap`
callback. On an empty collection the callback never runs, so the test passes having asserted
nothing. Assert the length first, or pull at least one check outside the iterator.

=== "BAD"
    ```javascript
    it('all valid', () => {
      rows.forEach(r => expect(r.valid).toBe(true));   // JS25 - nothing runs if rows is []
    });
    ```
=== "CLEAN"
    ```javascript
    it('all valid', () => {
      expect(rows.length).toBeGreaterThan(0);
      rows.forEach(r => expect(r.valid).toBe(true));
    });
    ```

### JS26 - fake timers installed but never advanced { #js26 }
`J1` · LOW · F2

`jest.useFakeTimers()` / `vi.useFakeTimers()` (or `sinon` fake timers) is set up, but the clock is
never advanced (`runAllTimers`, `advanceTimersByTime`, `tick`). The scheduled callback never fires,
so the assertion reads un-mutated state and passes for the wrong reason.

### JS27 - toHaveBeenCalled* is the sole oracle on a locally-created double { #js27 }
`J3` · LOW · F4

The only assertion is `toHaveBeenCalled` / `toHaveBeenCalledWith` / `toHaveBeenCalledTimes` on a
mock created in the test (`jest.fn()` / `vi.fn()`). It verifies the test's own wiring, that the
double was invoked, not that the unit produced the right result.

### JS29 - resolves / rejects chain is a bare statement { #js29 }
`J1` · LOW · F2

`expect(p).resolves.toBe(...)` / `.rejects...` written as a statement that is neither `await`ed
nor `return`ed. The matcher returns a promise; the test finishes green before it settles, so a
later rejection is lost.

=== "BAD"
    ```javascript
    it('resolves', () => {
      expect(load()).resolves.toBe(42);    // JS29 - not awaited or returned
    });
    ```
=== "CLEAN"
    ```javascript
    it('resolves', async () => {
      await expect(load()).resolves.toBe(42);
    });
    ```

### JS30 - literal-vs-literal assertion { #js30 }
`J2` · HIGH · F3

Both operands are fixed at parse time: `expect(2).toBe(3)`, chai `expect(1).to.equal(1)`. The
assertion does not touch the unit under test, so it is either always-true or always-false by
construction, never a check on real behaviour.

### JS31 - try/catch swallows a possible throw with no assertion on the exception { #js31 }
`J1` · LOW · F2

A `try` calls the unit and the `catch` neither re-throws nor asserts anything on the error. A unit
that stops throwing still passes green. Sibling of JS11, where the swallowed thing is an `expect`;
here there is no assertion at all on the caught path.

=== "BAD"
    ```javascript
    it('throws on bad input', () => {
      try { parse('bad'); } catch (e) { /* JS31 - swallowed, nothing asserted */ }
    });
    ```
=== "CLEAN"
    ```javascript
    it('throws on bad input', () => {
      expect(() => parse('bad')).toThrow(SyntaxError);
    });
    ```

## High-value traps with evidence

The scanner also catches a set of idiom-specific false-greens documented in the JS/TS empirical
studies:

- **HTTP header case trap** (`J4`): supertest lowercases response headers, so
  `res.headers['Content-Type']` is always `undefined`. Use the lowercase key.
- **Bitwise NOT coercion** (`J4`): `~~res.header['content-length']` turns an absent header into
  `0`, passing silently when the expected value is `0`.
- **Self-referential field oracle** (`J2`): `expect(result).toEqual([{ createdAt: result[0].createdAt }])`
  compares a field to itself. Use a fixed known value.
- **Sign-then-verify tautology** (`J2`): signing a JWT and verifying it with the same key passes
  even if `verify()` skips the check, unless paired with a wrong-key negative test.

## Project layer - config audit (PL)

Emitted by `--config-audit`, not the per-file scan. The suite goes green by runner configuration,
not by a smell inside any one test file.

### PL7 - no coverage gate { #pl7 }
`J5` · LOW · F8

No `coverageThreshold` / `coverage.thresholds` is set, so coverage can fall to zero and the suite
still passes. Add a coverage threshold.

### PL8 - `bail` stops the run early { #pl8 }
`J5` · LOW · F5

`bail` is set, so the run stops on the first failures and the reported test count is incomplete.

### PL10 - `passWithNoTests` lets an empty suite pass { #pl10 }
`J1` · LOW · F5

`passWithNoTests` lets an empty or fully-filtered suite report green, so a misconfigured glob or a
deleted file goes unnoticed.

## Diagnostic codes (opt-in, OFF by default) { #diagnostics }

Family F8: not false-green (the test still protects), shown only on a diagnostic pass.

| Code | What it flags |
|---|---|
| **D1** | assertion roulette: many assertions in one test, none identifying which failed |
| **D3** | duplicate assert: the same assertion appears more than once |
| **D4** | untitled `it.each` / `test.each` cases: a failing case is named only by index |
| **D6** | `console.*` in a test body: a debug artifact that bypasses the oracle |
| **D7** | anonymous test: empty or missing description |
| **D8** | magic number in an assertion: a bare numeric literal instead of a named constant |
| **M2** | over-long test body: exceeds the line-count threshold |

## Look-alikes: do NOT flag

- `expect(fn).not.toThrow()` after a setup call: the absence of an exception is the meaningful
  assertion.
- `expect(emitter).toHaveProperty('on')`: duck-typing check; interface presence is the contract.
- `expectTypeOf(v).toEqualTypeOf<T>()`: a compile-time type assertion; fails at `tsc`, not C5.
- `expect(result).toMatchObject({ kind: 'error' })` on a discriminated union: the discriminant
  determines the branch; not C6.
- `vi.mocked(fn)` / `jest.mocked(fn)`: typed mock wrappers, the TS equivalent of `autospec`; not
  C13b.
- `done()` after a series of real assertions: the smell is `done()` *before* the assertion, not
  after.
