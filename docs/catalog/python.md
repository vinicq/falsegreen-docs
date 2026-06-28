# Python catalog

The `C*` structural codes, as implemented by the [falsegreen](../scanners/python.md) scanner: a
zero-dependency AST pass over pytest and unittest. Each code names the signal it keys on and,
where it helps, the look-alike it deliberately leaves alone.

Confidence: **HIGH** blocks, **LOW** warns, **OFF** is diagnostic-only. Judgments are
[J1-J6](../concepts/judgments.md); families are [F1-F8](../concepts/taxonomy.md).

---

## Family A - the test never checks anything

Failure modes F1 (no oracle) and F2 (the check never runs).

### C1 - assertion inside a conditional or loop that may never run
`J1` · LOW · F2

The `assert` (or `self.assert*`) lives inside an `if`, `for`, or `while` whose condition could
be false or whose iterable could be empty. The test passes vacuously when the branch is never
entered.

!!! note "Signal"
    The assertion is not reachable from the function's top level without entering a conditional.
    Not flagged when the loop iterates a non-empty literal (`for x in (1, 2, 3):`).

=== "BAD"
    ```python
    def test_items():
        for item in items:        # items could be []
            assert item.valid     # never runs if items is empty
    ```
=== "CLEAN"
    ```python
    def test_items():
        assert len(items) > 0
        for item in items:
            assert item.valid
    ```

### C2 - test body contains no assertion at all
`J1` · HIGH · F1

No `assert`, no `self.assert*`, no `pytest.raises()`, no fluent `.should.`, no mock assertion.
The body is only `pass`, a docstring, `...`, or setup. Always green regardless of the code.

!!! note "Signal"
    No verification of any kind in the body. Exemptions: `@pytest.mark.skip`,
    `@pytest.mark.xfail`, and `@hypothesis` / `@given` / `@fuzz` decorators.

=== "BAD"
    ```python
    def test_create_user():
        user = create_user("Alice")   # no assert - always green
    ```

### C2b - test calls production code but verifies nothing
`J1` · LOW · F1

Like C2, but with real calls to the unit under test. The check is simply missing. Kept separate
because it is easy to mistake for a delegation pattern.

!!! note "Signal"
    A real SUT call with no assertion after it. Exemption: if the test calls a helper that
    itself contains the assertion, the check executes through the helper, so it is not flagged.

=== "BAD"
    ```python
    def test_process():
        result = process(data)        # calls SUT but no assert follows
    ```

### C2c - empty self.subTest block
`J1` · LOW · F1

A unittest `with self.subTest(...):` block that wraps work but contains no assertion - the subTest
analogue of an empty test, since each generated sub-case runs and verifies nothing. More specific
than C2b, which it suppresses for this shape. A subTest that asserts, raises, or delegates to a
`check_*`/`verify_*` helper is not flagged; the receiver must be `self`/`cls`.

=== "BAD"
    ```python
    for i in cases:
        with self.subTest(i=i):
            do_thing(i)               # no assertion inside the block
    ```

### C3 - assert inside a try whose except swallows the error
`J1` · HIGH · F2

A `try` contains an `assert`, and the `except` catches `AssertionError`, `Exception`, or bare
`except:` with a body that is only `pass` / `continue`. The failure is eaten; the test stays
green.

!!! note "Signal"
    Assertion inside `try`, handler swallows it. A handler that re-raises or does meaningful
    work is not C3.

=== "BAD"
    ```python
    def test_value():
        try:
            assert compute() == 42
        except Exception:
            pass                      # C3 - hides the failure
    ```

### C4 - test function not collected by pytest
`J1` · HIGH · F5

A `def test_*` defined nested inside another function or class method, with a real assertion,
never called and never decorated as a route or callback. pytest only collects top-level or
class-method tests; this one is invisible to the runner.

!!! note "Signal"
    Nested `test_*` with an assertion, no caller. Exemption: framework callbacks (`@app.get`,
    `@click.command`, awaited coroutines, route handlers) are not C4.

### C4b - test class has `__init__`
`J1` · LOW · F5

A class named `Test*` (or a `unittest.TestCase` subclass) defines `__init__`. pytest skips such
classes entirely, so none of its tests run.

### C20 - assertion after an unconditional return / raise / fail
`J1` · HIGH · F2

An `assert` appears after a `return`, `raise`, `break`, `continue`, or `pytest.fail()` in the
same block. Dead code; never reached. Detection uses structured intra-test (block-level)
reachability, so it catches an assertion after a return / raise / fail in any block, not just at
the top level.

=== "BAD"
    ```python
    def test_flag():
        if not flag:
            return
        assert flag          # reachable, ok
        return               # unconditional return
        assert True          # C20 - dead, never runs
    ```

### C21 - every assertion is inside a conditional; none runs unconditionally
`J1` · LOW · F2

The function has assertions, but every check is inside an `if` branch with no exhaustive
if/else that guarantees at least one runs. The test can pass without checking anything. The same
structured (block-level) reachability model decides this, so C21 fires only when no assertion sits
on the test's guaranteed spine.

### C22 - async test never awaits the unit under test
`J1` · OFF · F2

An `async def test_*` makes calls and has assertions but contains no `await`, `async with`,
`async for`, and does not drive a loop (`asyncio.run`, `run_until_complete`, `anyio.run`). The
coroutine may return before any I/O completes. Opt-in.

### CC - commented-out assert
`J1` · LOW · F2

A line in the body is `# assert ...`: a check that was commented out and left. The assertion
never runs. A strong signal the test was weakened.

=== "BAD"
    ```python
    def test_total():
        result = total(items)
        # assert result == 42    # CC - this check is disabled
    ```

---

## Family B - the check is weak or always true

Mostly F3 (the check is trivially true).

### C5 - always-true assertion
`J2` · HIGH · F3

The assertion is structurally guaranteed to pass: `assert True`, `assert (x, y)` (a non-empty
tuple is always truthy), `assert 1`, `assert x or True`. The check adds no protection.

=== "BAD"
    ```python
    def test_items():
        assert (item_a, item_b)   # C5 - non-empty tuple, always True
    ```

### C6 - weak assertion: only checks that something came back
`J4` · LOW · F4

The assertion checks only truthiness (`assert result`), non-empty length, or string containment
without verifying the actual value or structure.

!!! note "Signal"
    Truthiness or length-only check. Exemption: in web/browser tests, a truthy response or
    locator IS the contract. `assert response.status_code` in an HTTP test is not flagged.

=== "BAD"
    ```python
    def test_users():
        result = get_users()
        assert result            # C6 - only checks non-empty
    ```
=== "CLEAN"
    ```python
    def test_users():
        result = get_users()
        assert len(result) == 3
        assert result[0].name == "Alice"
    ```

### C6b - assertion on a positional mock argument via a computed index
`J3` · LOW · F4

The test reads `mock.call_args.args[idx]` or `mock.call_args[0][idx]` where `idx` is computed
(`.index()`, arithmetic, a variable) rather than a fixed literal. The position is fragile and
may silently shift.

### C6c - mock call_count truthiness as the oracle
`J4` · LOW · F4

`assert mock.call_count` (bare) passes on any count `>= 1`, so it checks only that the mock was
called, not how many times. The receiver must be a known mock; an exact or lower-bounded count
(`== N`, `>= 1`) is a real check. The always-true `mock.call_count >= 0` form is C44.

### C7 - self-comparison: both sides are identical
`J2` · HIGH · F3

`assert x == x`, `assertEqual(x, x)`, or any comparison where both sides are syntactically
identical and contain no function calls. Always true by reflexivity.

!!! note "Signal"
    Identical operands, no calls. Exemption: if the test also checks `x != peer`, `x in {x}`,
    or `hash(x)`, it is testing `__eq__` / `__hash__` semantics, not C7.

=== "BAD"
    ```python
    def test_name():
        name = get_name()
        assert name == name    # C7 - always true
    ```

### C8 - float exact equality
`J4` · LOW · F4

`==` against a non-sentinel float literal (anything other than `0.0` or `1.0`). Floating-point
arithmetic makes exact equality unreliable.

=== "BAD"
    ```python
    assert compute() == 3.14159    # C8
    ```
=== "CLEAN"
    ```python
    assert compute() == pytest.approx(3.14159, rel=1e-6)
    ```

### C8b - approximate equality with no explicit tolerance
`J4` · LOW · F4

`assertAlmostEqual`/`assertNotAlmostEqual` (default 7 places) or `== pytest.approx(...)` (default
1e-6 relative) with no `places=`/`delta=`/`rel=`/`abs=`. The default tolerance can pass a
meaningfully wrong value. Sizing the tolerance to the values keeps it quiet.

=== "BAD"
    ```python
    self.assertAlmostEqual(total(), 4.2)      # default 7 places
    assert total() == pytest.approx(4.2)      # default 1e-6 rel
    ```
=== "CLEAN"
    ```python
    self.assertAlmostEqual(total(), 4.2, places=2)
    ```

### C9 - pytest.raises too broad
`J4` · LOW · F4

`pytest.raises()` with no exception type, or a very broad one (`Exception`, `BaseException`) and
no `match=`. Any exception, including one from a typo inside the test, satisfies the check.

=== "BAD"
    ```python
    with pytest.raises(Exception):   # C9 - anything passes
        divide(a, b)
    ```
=== "CLEAN"
    ```python
    with pytest.raises(ZeroDivisionError, match="division by zero"):
        divide(a, 0)
    ```

### C11a - self-confirming literal: assigns then asserts the same value
`J2` · LOW · F3

`obj.attr = VALUE` followed by `assert obj.attr == VALUE` with the same literal. The test
confirms Python's attribute assignment works, not the production code.

=== "BAD"
    ```python
    def test_price():
        product.price = 100
        assert product.price == 100   # C11a - just confirms assignment
    ```

### C13 - mock assertion misspelled or not called
`J4` · HIGH · F2

A mock assertion accessed as an attribute without `()`: `mock.assert_called_once` instead of
`mock.assert_called_once_with()`. The attribute access returns a bound method; the check never
runs. Also flags invented names (`assert_called_twice`, `called_once_with`).

=== "BAD"
    ```python
    mock_fn.assert_called_once      # C13 - missing (), does nothing
    ```
=== "CLEAN"
    ```python
    mock_fn.assert_called_once_with(expected_arg)
    ```

### C13b - patch() without autospec
`J3` · LOW · F4

`@patch('module.Thing')` or `patch.object(obj, 'method')` without `autospec=True`, `spec=`, or
`spec_set=`. The mock accepts any call signature silently; typos in argument names or counts go
undetected.

### C14 - golden file generated from the actual output
`J2` · LOW · F3

`if not exists(golden_path): write(golden_path, actual_output)`. On the first run the test
writes the current (possibly wrong) output as the expected value, then compares against it
forever.

!!! note "Signal"
    Write-if-missing on a golden path. Exemption: in browser snapshot testing (Playwright,
    Selenium) this is intentional and not flagged.

### C16 - result depends on uncontrolled time, randomness, or sleep
`J6` · LOW · F6

`time.sleep(N)`, `datetime.now()` / `time.time()` without `freezegun` / `time_machine`,
`random.*` without `random.seed()`, `torch.rand*` without `torch.manual_seed()`, or
`train_test_split` without `random_state=`. Also flags `uuid.uuid4()` / `uuid.uuid1()` /
`uuid.getnode()` and `secrets.token_*` / `secrets.randbits` / `secrets.choice`, all
module-qualified. A bare `from uuid import uuid4` call and the deterministic `uuid.uuid5()` are
not flagged.

=== "BAD"
    ```python
    def test_expiry():
        created = datetime.now()      # C16 - not frozen
        assert is_expired(created, ttl=0) is False
    ```

### C18 - string / repr comparison
`J2` · LOW · F4

`==` where one side is `str(x)`, `repr(x)`, `format(x, ...)`, or an f-string, against a string
literal. The string format is an implementation detail; it changes without a semantic change.

=== "BAD"
    ```python
    assert str(user) == "User(Alice, 30)"   # C18 - couples to str() format
    ```
=== "CLEAN"
    ```python
    assert user.name == "Alice" and user.age == 30
    ```

### C25 - xfail without strict=True
`J1` · LOW · F5

`@pytest.mark.xfail` without `strict=True`. If the test unexpectedly passes, pytest reports
`XPASS`, not a failure. A quietly passing xfail hides that the bug was fixed without removing
the mark.

### C34 - suboptimal assertion form
`J4` · LOW · F8

`assert not x in y` (use `x not in y`), `assert len(x) == 0` (use `assert not x`),
`assert x == True` / `== False` / `== None` / `!= None` (use `is` / truthiness). These weaken
the error message and obscure intent.

---

## Family C - the test checks its own setup, not the program

### C19 - pytest.raises wraps more than one call
`J1` · LOW · F4

A `with pytest.raises(E):` block holds more than one statement. If the first raises, the second
never runs, so the test may be checking a different line than intended.

=== "BAD"
    ```python
    with pytest.raises(ValueError):
        setup_data()          # this might raise, not the SUT
        sut.process(data)     # C19 - intended target
    ```

### C28 - pytest.raises binding variable never read
`J4` · LOW · F4

`with pytest.raises(E) as exc:` where `exc` is never used afterward. The exception type is
checked but not its message or attributes.

=== "BAD"
    ```python
    with pytest.raises(ValueError) as exc:   # C28 - exc never read
        process(bad_input)
    ```
=== "CLEAN"
    ```python
    with pytest.raises(ValueError) as exc:
        process(bad_input)
    assert "must be positive" in str(exc.value)
    ```

### C29 - os.environ modified directly in a test
`J6` · LOW · F6

`os.environ["KEY"] = value`, `os.environ.update(...)`, or `os.putenv(...)` in a test body. The
change persists across tests in the same process. Use `monkeypatch.setenv()`.

---

## Family D - the test depends on external or shared state

Mostly F6 (passes or fails by luck or by order).

### C17 - pytest.skip() inside a broad except
`J1` · HIGH · F5

A `try` with an assertion, where the `except` is broad and calls `pytest.skip()` or
`skipTest()`. A real failure triggers the skip instead of failing the test. Green even when the
SUT is broken.

=== "BAD"
    ```python
    def test_api():
        try:
            assert fetch_data() == expected
        except Exception:
            pytest.skip("skipping")   # C17 - hides real failures
    ```

### C23 - hard-coded absolute or home-relative file path
`J6` · LOW · F6

`open("/home/user/data.csv")` or `Path("/tmp/fixture.json").read_text()`. The path does not
exist in CI or on another machine. Use `tmp_path` or `Path(__file__).parent / "data.csv"`.

### C24 - module-level mutable state mutated by a test
`J6` · LOW · F6

The module declares a global `list`, `dict`, or `set`; a test mutates it with no autouse
fixture resetting it. Test order decides the outcome.

=== "BAD"
    ```python
    _cache = {}                       # module-level mutable

    def test_fill():
        _cache["key"] = "value"       # C24 - mutates shared state

    def test_read():
        assert _cache["key"] == "value"  # passes only after test_fill
    ```

### C27 - try/except/pass around a SUT call with no assertion
`J1` · HIGH · F1

A `try` calls the SUT with no assertion, and the `except` is `pass`-only. Success and failure
both go green. Different from C3, which wraps an assert; C27 has no assert at all.

=== "BAD"
    ```python
    def test_process():
        try:
            process(data)      # C27 - success and failure both -> green
        except Exception:
            pass
    ```

### C30 - HTTP mock not activated
`J3` · LOW · F4

`responses.add(...)` or `httpretty.register_uri(...)` called, but the activator
(`@responses.activate`, `responses.start()`, `httpretty.enable()`) is absent. Real HTTP goes
through; the mock is never used.

### C31 - capsys.readouterr() result discarded
`J4` · LOW · F1

`capsys.readouterr()` called as a bare expression, or assigned to a variable never read. The
capture ran but nothing was checked.

=== "BAD"
    ```python
    def test_output(capsys):
        run()
        capsys.readouterr()          # C31 - captured but never asserted
    ```
=== "CLEAN"
    ```python
    def test_output(capsys):
        run()
        out, _ = capsys.readouterr()
        assert out == "hello\n"
    ```

### C32 - @pytest.mark.skip without reason
`J1` · LOW · F5

`@pytest.mark.skip` with no `reason=`. No explanation for the disabled test; may be forgotten
permanently.

### C35 - retry / flaky decorator
`J6` · LOW · F6

A decorator named `flaky`, `repeat`, `retry`, `rerun`, or `flake` on a test. Masks
non-determinism instead of fixing it.

---

## Family E - it passes, but checks the wrong thing

### C33 - ML metric computed but not asserted
`J4` · LOW · F1

An sklearn metric (`accuracy_score`, `f1_score`, `model.score()`) whose result is discarded or
assigned to a variable never read. The metric was computed but never validated against a
threshold.

=== "BAD"
    ```python
    def test_model():
        acc = accuracy_score(y_true, y_pred)   # C33 - never asserted
    ```
=== "CLEAN"
    ```python
    def test_model():
        acc = accuracy_score(y_true, y_pred)
        assert acc >= 0.90
    ```

### C36 - pytest.fail() without reason
`J1` · LOW · F8

`pytest.fail()` with no message. The failure is unintelligible in CI output.

### C37 - duplicate parametrize case
`J2` · LOW · F8

`@pytest.mark.parametrize` where the same argument set appears twice. The duplicate confirms the
same code path again and adds no coverage.

---

## Family additions (catalog sync)

### C38 - two tests share a name
`J1` · HIGH · F5

Two `def test_*` at module or class scope with the same name. Python binds the later over the
earlier, so the first never runs.

### C39 - returns a comparison instead of asserting
`J1` · HIGH · F1

`return x == y` in a test. pytest ignores the returned value (it warns with
`PytestReturnNotNoneWarning`); nothing is checked.

### C41 - assertion on a None-returning mutator
`J4` · LOW · F3

`assert not lst.sort()` / `assertIsNone(lst.sort())`. Whether it is trivially green depends on
the receiver's type, so this is a skill-only judgment, restricted to known mutators
(`sort`, `append`, `extend`, `reverse`, `update`, `add`, `remove`, `insert`, `clear`).

### C42 - assertion on a generator or lambda
`J2` · HIGH · F3

`assert (x for x in y)` / `assert lambda: ...`. The object is always truthy. A list, set, or
dict comprehension is not C42, because it can be empty.

### C43 - mid-test skip
`J1` · LOW · F5

`pytest.skip()` after test logic, with checks below it that then never run. A skip at the top is
a legitimate guard.

### C44 - numeric tautology
`J2` · HIGH · F3

`len(x) >= 0`, `abs(x) >= 0`, `len(x) > -1`, or a mock's `call_count >= 0` / `> -1`. The
comparison is always true.

### C45 - empty parametrize
`J1` · HIGH · F5

`@pytest.mark.parametrize("...", [])`. Zero cases are generated, so the test never runs.

### C48 - dark patch: flips a test-mode flag then asserts
`J1` · LOW · F2

The test forces a test-mode toggle into test mode (`os.environ["TESTING"] = "1"`,
`settings.TESTING = True`, a `global`-declared `TESTING = True`) and then asserts, so it exercises
the product's test-only branch (`if TESTING: ...`) instead of real behaviour.

!!! note "Signal"
    A known test-mode flag flipped on before the assertion. Does not fire when a genuine assertion
    already runs before the flip, unless a post-flip assertion reads the toggled flag itself.
    Config values and product feature flags are not flagged.

=== "BAD"
    ```python
    def test_login():
        os.environ["TESTING"] = "1"   # C48 - forces the test-only branch
        assert login(user, pwd) is True
    ```

---

## Diagnostic codes (opt-in, OFF by default)

Family F8: not false-green (the test still protects), shown only on a diagnostic pass. Dedicated
linters (ruff) also cover these.

| Code | What it flags |
|---|---|
| **D1** | assertion roulette: two or more asserts, none with a message |
| **D3** | duplicate assert: the identical `assert` appears twice |
| **D4** | unnamed parametrize: 3+ cases with no `ids=` |
| **D5** | excessive inline setup: more than 5 statements before the first assert |
| **D6** | debug `print()` left in the test body |
| **M2** | long test method: body over 50 lines |

---

## Look-alikes: do NOT flag

These resemble a smell but are correct. The scanner leaves them alone.

- `@pytest.mark.skip` / `@pytest.mark.xfail` on an empty body: explicitly disabled, not C2.
- `@given` / `@hypothesis` / `@fuzz` with no explicit `assert`: hypothesis asserts internally,
  not C2.
- A helper called from the test that holds the `assert`: not C2b.
- `for x in (1, 2, 3): assert x`: not C1, the literal is non-empty.
- `assert response` in an HTTP test / `assert locator` in a Playwright test: not C6, presence is
  the assertion at that layer.
- `assert x == x` where the test also checks `x != peer` or `hash(x)`: testing `__eq__` /
  `__hash__`, not C7.
- `freezegun` / `time_machine` imported: an unfrozen `datetime.now()` is not C16.
- `patch(..., autospec=True)`: not C13b.
- `with pytest.raises(E) as exc: ...; assert "msg" in str(exc.value)`: exc is read, not C28.
