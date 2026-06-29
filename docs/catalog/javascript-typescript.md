# JavaScript / TypeScript catalog

The codes implemented by [falsegreen-js](../scanners/javascript.md): a static scan via the
TypeScript compiler API, runner-agnostic across Jest, Vitest, Mocha+Chai, Jasmine, AVA,
node:test, Cypress, Playwright, and Testing Library. Covers `.js`, `.jsx`, `.ts`, `.tsx`,
`.mjs`, `.cjs`, `.mts`, `.cts`.

Codes share an id with [Python](python.md) where the concept matches; `JS*` codes are
ecosystem-specific. Confidence: **HIGH** blocks, **LOW** warns. Judgments are
[J1-J6](../concepts/judgments.md).

## Shared codes (same concept as Python)

These mirror the [Python catalog](python.md) entries; the signal is the JavaScript form.

| Code | Conf | JavaScript form |
|---|---|---|
| C2 | HIGH | empty test body |
| C2b | LOW | calls the unit but never asserts |
| C5 | HIGH | always-true (`expect(true).toBe(true)`, `assert(1)`) |
| C6 | LOW | weak check (`toBeTruthy`/`toBeDefined`, `.length > 0`) |
| C7 | HIGH | self-compare (`expect(x).toBe(x)`) |
| C8 | LOW | exact equality on a float |
| C8b | LOW | `toBeCloseTo` with no precision argument — the default 2-digit tolerance may be too loose |
| C9 | LOW | `toThrow()` with no error type or message |
| C11a | LOW | self-confirming literal — the expected value is bound from the same call under test (`const e = foo(); expect(foo()).toBe(e)`) |
| C16 | LOW | depends on `Date.now` / `new Date()` / `Math.random` / `crypto.randomUUID`/`getRandomValues` / a fixed timer |
| C18 | LOW | stringified equality (`String(x)` / `JSON.stringify` / template literal) |
| C20 | HIGH | dead assertion after `return` / `throw` / `process.exit` / an exhaustive `switch` (structured block-level reachability, cfg.ts) |
| C21 | LOW | no assertion runs unconditionally; a dead assertion does not mask it (same cfg.ts reachability) |
| C23 | LOW | reads a real file at a literal path / hard-coded URL |
| C37 | LOW | duplicate `it.each` / `test.each` case |
| C44 | HIGH | numeric tautology on a length (`expect(x.length).toBeGreaterThanOrEqual(0)`) |
| C48 | LOW | dark patch: flips a test-mode flag (`process.env.NODE_ENV = "test"`, `process.env.TESTING = "1"`, `settings.TESTING = true`) then asserts (parity with Python `C48`) |
| CC | LOW | commented-out assertion |

## JavaScript-specific codes

### JS1 - focused test skips the rest of the suite
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

### JS2 - expect with no matcher
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

### JS3 - snapshot is the only assertion
`J4` · LOW · F3

A test whose only check is `toMatchSnapshot()` / `toMatchInlineSnapshot()`. The baseline is
generated from the output, so it detects change, not correctness.

### JS4 - skipped test
`J1` · LOW · F5

`it.skip` / `xit` / `it.todo` left in place. Silently excluded from the run.

### JS5 - async query or event not awaited
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

### JS6 - empty describe / suite
`J1` · HIGH · F1

A `describe`/`suite` block with no test inside. It contributes nothing and can read as coverage.

### JS7 - assertion in a non-awaited setTimeout / then callback
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

### JS8 - mocks the unit under test and asserts it directly
`J3` · LOW · F4

The test mocks the very function it claims to test, then asserts the mock's value. It tests the
mock configuration, not the code. (The semantic, deeper form is case 10 / `S12`.)

### JS9 - assertion in a dead literal branch
`J1` · HIGH · F2

`if (false) { expect(...) }` or the `else` of `if (true)`. The assertion is unreachable.

### JS11 - try/catch swallows the assertion
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

### JS13 - query used as a loose statement, never asserted
`J4` · LOW · F1

`getBy*` / `queryBy*` called as a bare statement. The query runs but nothing is checked on it.

### JS15 - comparison wrapped in a boolean
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

### JS17 - commented-out test block
`J1` · LOW · F2

`// it('...', ...)` left in the file. The test does not run.

### JS18 - done callback instead of async/await
`J1` · LOW · F2

A `done`-callback test where `done()` precedes or sits at the same level as the assertion, so the
runner completes before the assertion throws.

### JS21 - matcher referenced but never called
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

### JS22 - empty it.each / test.each table
`J1` · HIGH · F5

`it.each([])(...)`. Zero cases are generated, so the test never runs. Parallel of `C45`.

### JS23 - expect.assertions(N) the body cannot satisfy
`J1` · HIGH · F5

`expect.assertions(N)` with a numeric `N` higher than the unconditional, reachable, non-nested
`expect()` calls that can run. The guard the author wrote to defend against a silently-dropped
async assertion is dead on arrival. Fires only on a provable literal-`N` shortfall: an `expect`
in a loop, a branch, a `.then`/callback, or a called helper makes the count indeterminate and
suppresses the finding. `expect.hasAssertions()` carries no count and is skipped.

### JS24 - Cypress query with no assertion
`J4` · LOW · F4

A Cypress query chain (`cy.get`/`cy.find`/`cy.contains`) used as a statement with no terminating
`.should`/`.and` and no `expect` inside a `.then` callback. The query produces a subject that is
never asserted, the cy.* analogue of JS13. Action commands (`click`/`type`/`visit`/...) do work
rather than query, so a chain ending in one stays clean, as does one ending in `.should`/`.and`.

### JS25 - the only assertion is inside an array-iterator callback
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

### JS26 - fake timers installed but never advanced
`J1` · LOW · F2

`jest.useFakeTimers()` / `vi.useFakeTimers()` (or `sinon` fake timers) is set up, but the clock is
never advanced (`runAllTimers`, `advanceTimersByTime`, `tick`). The scheduled callback never fires,
so the assertion reads un-mutated state and passes for the wrong reason.

### JS27 - toHaveBeenCalled* is the sole oracle on a locally-created double
`J3` · LOW · F4

The only assertion is `toHaveBeenCalled` / `toHaveBeenCalledWith` / `toHaveBeenCalledTimes` on a
mock created in the test (`jest.fn()` / `vi.fn()`). It verifies the test's own wiring, that the
double was invoked, not that the unit produced the right result.

### JS29 - resolves / rejects chain is a bare statement
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

### JS30 - literal-vs-literal assertion
`J2` · HIGH · F3

Both operands are fixed at parse time: `expect(2).toBe(3)`, chai `expect(1).to.equal(1)`. The
assertion does not touch the unit under test, so it is either always-true or always-false by
construction, never a check on real behaviour.

### JS31 - try/catch swallows a possible throw with no assertion on the exception
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

## Diagnostic codes (opt-in, OFF by default)

Family F8: not false-green. `D1` assertion roulette, `D3` duplicate assert, `D4` untitled
`it.each` cases, `D6` `console.*` in a test, `D7` anonymous test, `D8` magic number in an
assertion, `M2` over-long test body.

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
