# Python catalog

The `C*` structural codes, as implemented by the [falsegreen](../scanners/python.md) scanner: a
zero-dependency AST pass over pytest and unittest. Each code names the signal it keys on and,
where it helps, the look-alike it deliberately leaves alone.

Confidence: **HIGH** blocks, **LOW** warns, **OFF** is diagnostic-only. Judgments are
[J1-J6](../concepts/judgments.md); families are [F1-F8](../concepts/taxonomy.md).

Every emitted code has its own entry below, grouped by family. The index links straight to each
one.

## Index

| Code | Conf | J | One-liner |
|---|---|---|---|
| [C1](#c1) | LOW | J1 | assert inside an if/for that may never run |
| [C2](#c2) | HIGH | J1 | no check at all (empty body) |
| [C2b](#c2b) | LOW | J1 | calls things but checks nothing |
| [C2c](#c2c) | LOW | J1 | empty `self.subTest(...)` block |
| [C3](#c3) | HIGH | J1 | assert inside a try whose except swallows the error |
| [C4](#c4) | HIGH | J1 | not collected by pytest (never runs) |
| [C4b](#c4b) | LOW | J1 | test class has `__init__` (not collected) |
| [C5](#c5) | HIGH | J2 | always-true check (`assert True` / tuple / `or True`) |
| [C6](#c6) | LOW | J4 | weak check (only that something came back) |
| [C6b](#c6b) | LOW | J5 | coupled to positional argument layout |
| [C6c](#c6c) | LOW | J4 | `call_count` truthiness as the oracle |
| [C7](#c7) | HIGH | J2 | compares a thing to itself |
| [C8](#c8) | LOW | J4 | exact equality on a float |
| [C8b](#c8b) | LOW | J4 | approximate equality with no explicit tolerance |
| [C9](#c9) | LOW | J4 | `pytest.raises` too broad |
| [C11a](#c11a) | LOW | J2 | self-confirming literal assigned by the test |
| [C13](#c13) | HIGH | J3 | mock assertion misspelled / not called |
| [C13b](#c13b) | LOW | J3 | `patch()` without autospec |
| [C14](#c14) | LOW | J2 | golden/snapshot generated from the output |
| [C16](#c16) | LOW | J1 | depends on time, randomness, or a fixed sleep |
| [C17](#c17) | HIGH | J1 | skip inside a broad except hides a failure |
| [C18](#c18) | LOW | J2 | compares `str()`/`repr()` to a literal |
| [C19](#c19) | LOW | J1 | `pytest.raises` wraps more than one call |
| [C20](#c20) | HIGH | J1 | assertion in dead code after return/raise/fail |
| [C21](#c21) | LOW | J1 | every assertion is conditional, none runs |
| [C22](#c22) | OFF | J1 | async test asserts but never awaits the unit |
| [C23](#c23) | LOW | J6 | opens a real file at a literal path |
| [C24](#c24) | LOW | J6 | module-global mutable state shared across tests |
| [C25](#c25) | LOW | J4 | xfail without `strict=True` (XPASS treated as pass) |
| [C27](#c27) | HIGH | J1 | try/except/pass with no assertion at all |
| [C28](#c28) | LOW | J4 | `pytest.raises` binding never inspected |
| [C29](#c29) | LOW | J6 | `os.environ` assigned directly (leaks between tests) |
| [C30](#c30) | LOW | J3 | HTTP interceptor registered but not activated |
| [C31](#c31) | LOW | J4 | `capsys`/`capfd` output captured but not asserted |
| [C32](#c32) | LOW | J1 | `@pytest.mark.skip` without `reason=` |
| [C33](#c33) | LOW | J4 | sklearn metric computed but not asserted |
| [C34](#c34) | LOW | J4 | suboptimal assert form |
| [C35](#c35) | LOW | J1 | retry/flaky decorator masks flakiness |
| [C36](#c36) | LOW | J4 | `pytest.fail()` without a reason |
| [C37](#c37) | LOW | J4 | duplicate `parametrize` case |
| [C38](#c38) | HIGH | J1 | two tests share a name (the first never runs) |
| [C39](#c39) | HIGH | J1 | returns a comparison instead of asserting it |
| [C41](#c41) | LOW | J4 | assertion on a None-returning in-place mutator |
| [C42](#c42) | HIGH | J2 | assertion on a generator expression or lambda |
| [C43](#c43) | LOW | J1 | `pytest.skip()` after test logic |
| [C44](#c44) | HIGH | J2 | numeric tautology (`len()`/`abs()` always true) |
| [C45](#c45) | HIGH | J1 | empty `parametrize` list (zero cases) |
| [C48](#c48) | LOW | J1 | dark patch: flips a test-mode flag then asserts |
| [C49](#c49) | LOW | J1 | `pytest.warns`/`assertWarns` wraps more than one call |
| [C50](#c50) | LOW | J4 | `caplog`/`assertLogs` captured but not asserted |
| [C51](#c51) | HIGH | J1 | empty-bodied `pytest.raises`/`warns` context |
| [C52](#c52) | LOW | J2 | membership self-confirmation (`x in {x}`) |
| [C55](#c55) | LOW | J3 | compares two mock-rooted values |
| [C56](#c56) | LOW | J1 | sync assert of a never-awaited coroutine |
| [C57](#c57) | LOW | J3 | compares against an unconfigured Mock attribute |
| [C59](#c59) | HIGH | J1 | bare top-level comparison statement (loose-statement sibling of C39) |
| [CC](#cc) | LOW | J1 | commented-out assert |
| [D1](#diagnostics) | OFF | J4 | assertion roulette |
| [D3](#diagnostics) | OFF | J4 | duplicate assert |
| [D4](#diagnostics) | OFF | J4 | unnamed `parametrize` |
| [D5](#diagnostics) | OFF | J5 | excessive inline setup |
| [D6](#diagnostics) | OFF | J4 | debug `print()` in the body |
| [M2](#diagnostics) | OFF | J5 | over-long test method |
| [PL1](#pl1) | LOW | J1 | `-O`/`PYTHONOPTIMIZE` strips every assert |
| [PL2](#pl2) | LOW | J1 | `filterwarnings` does not promote warnings to errors |
| [PL7](#pl7) | LOW | J5 | no coverage gate |
| [PL8](#pl8) | LOW | J5 | `addopts` stops the run early |

A code keeps its id across languages where the smell is the same. Two ids carry a different
meaning per language, documented on each page: **C31** is the `capsys`/`capfd` capture here, but
the never-used captured value (`${x}= Get Text`) in [Robot](robot.md#c31); **C44** is the
numeric tautology here, widened in Robot to any vacuous library assertion (same id, broader
bucket).

---

## Family A - the test never checks anything

Failure modes F1 (no oracle) and F2 (the check never runs).

### C1 - assertion inside a conditional or loop that may never run { #c1 }
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

### C2 - test body contains no assertion at all { #c2 }
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

### C2b - test calls production code but verifies nothing { #c2b }
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

### C2c - empty self.subTest block { #c2c }
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

### C3 - assert inside a try whose except swallows the error { #c3 }
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

### C4 - test function not collected by pytest { #c4 }
`J1` · HIGH · F5

A `def test_*` defined nested inside another function or class method, with a real assertion,
never called and never decorated as a route or callback. pytest only collects top-level or
class-method tests; this one is invisible to the runner.

!!! note "Signal"
    Nested `test_*` with an assertion, no caller. Exemption: framework callbacks (`@app.get`,
    `@click.command`, awaited coroutines, route handlers) are not C4.

### C4b - test class has `__init__` { #c4b }
`J1` · LOW · F5

A class named `Test*` (or a `unittest.TestCase` subclass) defines `__init__`. pytest skips such
classes entirely, so none of its tests run.

### C20 - assertion after an unconditional return / raise / fail { #c20 }
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

### C21 - every assertion is inside a conditional; none runs unconditionally { #c21 }
`J1` · LOW · F2

The function has assertions, but every check is inside an `if` branch with no exhaustive
if/else that guarantees at least one runs. The test can pass without checking anything. The same
structured (block-level) reachability model decides this, so C21 fires only when no assertion sits
on the test's guaranteed spine.

### C22 - async test never awaits the unit under test { #c22 }
`J1` · OFF · F2

An `async def test_*` makes calls and has assertions but contains no `await`, `async with`,
`async for`, and does not drive a loop (`asyncio.run`, `run_until_complete`, `anyio.run`). The
coroutine may return before any I/O completes. Opt-in.

### CC - commented-out assert { #cc }
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

### C5 - always-true assertion { #c5 }
`J2` · HIGH · F3

The assertion is structurally guaranteed to pass: `assert True`, `assert (x, y)` (a non-empty
tuple is always truthy), `assert 1`, `assert x or True`. The check adds no protection.

=== "BAD"
    ```python
    def test_items():
        assert (item_a, item_b)   # C5 - non-empty tuple, always True
    ```

### C6 - weak assertion: only checks that something came back { #c6 }
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

### C6b - assertion on a positional mock argument via a computed index { #c6b }
`J3` · LOW · F4

The test reads `mock.call_args.args[idx]` or `mock.call_args[0][idx]` where `idx` is computed
(`.index()`, arithmetic, a variable) rather than a fixed literal. The position is fragile and
may silently shift.

### C6c - mock call_count truthiness as the oracle { #c6c }
`J4` · LOW · F4

`assert mock.call_count` (bare) passes on any count `>= 1`, so it checks only that the mock was
called, not how many times. The receiver must be a known mock; an exact or lower-bounded count
(`== N`, `>= 1`) is a real check. The always-true `mock.call_count >= 0` form is C44.

### C7 - self-comparison: both sides are identical { #c7 }
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

### C8 - float exact equality { #c8 }
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

### C8b - approximate equality with no explicit tolerance { #c8b }
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

### C9 - pytest.raises too broad { #c9 }
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

### C11a - self-confirming literal: assigns then asserts the same value { #c11a }
`J2` · LOW · F3

`obj.attr = VALUE` followed by `assert obj.attr == VALUE` with the same literal. The test
confirms Python's attribute assignment works, not the production code.

=== "BAD"
    ```python
    def test_price():
        product.price = 100
        assert product.price == 100   # C11a - just confirms assignment
    ```

### C52 - membership self-confirmation { #c52 }
`J2` · LOW · F3

`assert x in {x}` (or `x in [x]`, `x in (x,)`): the collection is built from the subject under
test, so membership holds by construction. A membership variant of C7. Checking against a
collection assembled independently of the subject is a real check.

=== "BAD"
    ```python
    def test_tag():
        tag = get_tag()
        assert tag in {tag}          # C52 - true by construction
    ```

### C13 - mock assertion misspelled or not called { #c13 }
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

### C13b - patch() without autospec { #c13b }
`J3` · LOW · F4

`@patch('module.Thing')` or `patch.object(obj, 'method')` without `autospec=True`, `spec=`, or
`spec_set=`. The mock accepts any call signature silently; typos in argument names or counts go
undetected.

### C14 - golden file generated from the actual output { #c14 }
`J2` · LOW · F3

`if not exists(golden_path): write(golden_path, actual_output)`. On the first run the test
writes the current (possibly wrong) output as the expected value, then compares against it
forever.

!!! note "Signal"
    Write-if-missing on a golden path. Exemption: in browser snapshot testing (Playwright,
    Selenium) this is intentional and not flagged.

### C16 - result depends on uncontrolled time, randomness, or sleep { #c16 }
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

### C18 - string / repr comparison { #c18 }
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

### C25 - xfail without strict=True { #c25 }
`J1` · LOW · F5

`@pytest.mark.xfail` without `strict=True`. If the test unexpectedly passes, pytest reports
`XPASS`, not a failure. A quietly passing xfail hides that the bug was fixed without removing
the mark.

### C34 - suboptimal assertion form { #c34 }
`J4` · LOW · F8

`assert not x in y` (use `x not in y`), `assert len(x) == 0` (use `assert not x`),
`assert x == True` / `== False` / `== None` / `!= None` (use `is` / truthiness). These weaken
the error message and obscure intent.

---

## Family C - the test checks its own setup, not the program

### C19 - pytest.raises wraps more than one call { #c19 }
`J1` · LOW · F4

A `with pytest.raises(E):` block holds more than one statement. If the first raises, the second
never runs, so the test may be checking a different line than intended.

=== "BAD"
    ```python
    with pytest.raises(ValueError):
        setup_data()          # this might raise, not the SUT
        sut.process(data)     # C19 - intended target
    ```

### C49 - pytest.warns / assertWarns wraps more than one call { #c49 }
`J1` · LOW · F4

A `with pytest.warns(W):` / `assertWarns` / `deprecated_call()` block holds more than one
statement. An unrelated earlier line may emit the warning while the target never does, so the
test passes without exercising the warning under test. The warns sibling of C19.

### C28 - pytest.raises binding variable never read { #c28 }
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

### C51 - empty-bodied pytest.raises / warns context { #c51 }
`J1` · HIGH · F1

`with pytest.raises(E):` (or `pytest.warns`) whose body is empty (`pass`, `...`, a comment).
No call is made inside the block, so the call that should raise never runs and the context
manager has nothing to catch. Always green.

=== "BAD"
    ```python
    with pytest.raises(ValueError):
        pass                          # C51 - nothing called, nothing raised
    ```

### C29 - os.environ modified directly in a test { #c29 }
`J6` · LOW · F6

`os.environ["KEY"] = value`, `os.environ.update(...)`, or `os.putenv(...)` in a test body. The
change persists across tests in the same process. Use `monkeypatch.setenv()`.

### C55 - comparison between two mock-rooted values { #c55 }
`J3` · LOW · F4

`assert m.foo == m.bar` where both operands derive from the same test double (a `Mock`,
`MagicMock`, or a `patch`-injected object). Each side is the test's own configured value, so
the comparison checks the doubles against each other, not the SUT.

### C56 - sync assert of a never-awaited coroutine { #c56 }
`J1` · LOW · F2

The asserted expression calls a local `async def` without awaiting it, so the operand is a
coroutine object, not its value. A coroutine is always truthy and never equals the expected
value the author had in mind; the real call never ran. Resolved file-wide against the set of
`async def` names.

=== "BAD"
    ```python
    async def fetch(): ...

    def test_fetch():
        assert fetch() == {"ok": True}   # C56 - asserts the coroutine, not its result
    ```
=== "CLEAN"
    ```python
    def test_fetch():
        assert asyncio.run(fetch()) == {"ok": True}
    ```

### C57 - assertion compares against an unconfigured Mock attribute { #c57 }
`J3` · LOW · F4

One side of the comparison is `m.attr` where `m` is a bare `Mock()` / `MagicMock()` with no
`spec=`/`spec_set=` and no assignment to `m.attr` in the body. Attribute access auto-creates a
fresh, truthy child Mock, so the expected side is the test's own auto-mock, not a real value.
Only the single-attribute shape (`m.attr`); both-sides-mock is C55's territory.

=== "BAD"
    ```python
    def test_role():
        m = Mock()
        assert user.role == m.role     # C57 - m.role is an auto-created Mock
    ```
=== "CLEAN"
    ```python
    def test_role():
        assert user.role == "admin"
    ```

---

## Family D - the test depends on external or shared state

Mostly F6 (passes or fails by luck or by order).

### C17 - pytest.skip() inside a broad except { #c17 }
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

### C23 - hard-coded absolute or home-relative file path { #c23 }
`J6` · LOW · F6

`open("/home/user/data.csv")` or `Path("/tmp/fixture.json").read_text()`. The path does not
exist in CI or on another machine. Use `tmp_path` or `Path(__file__).parent / "data.csv"`.

### C24 - module-level mutable state mutated by a test { #c24 }
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

### C27 - try/except/pass around a SUT call with no assertion { #c27 }
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

### C30 - HTTP mock not activated { #c30 }
`J3` · LOW · F4

`responses.add(...)` or `httpretty.register_uri(...)` called, but the activator
(`@responses.activate`, `responses.start()`, `httpretty.enable()`) is absent. Real HTTP goes
through; the mock is never used.

### C31 - capsys.readouterr() result discarded { #c31 }
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

### C50 - caplog / assertLogs output captured but never asserted { #c50 }
`J4` · LOW · F1

`caplog` is read (`caplog.records`, `caplog.text`) or `self.assertLogs(...)` is entered, but the
captured output is never asserted: no comparison on the records, messages, or levels. The capture
ran and had no effect on pass/fail. The logging sibling of C31.

=== "BAD"
    ```python
    def test_logs(caplog):
        run()
        caplog.records               # C50 - captured but never asserted
    ```
=== "CLEAN"
    ```python
    def test_logs(caplog):
        run()
        assert "started" in caplog.text
    ```

### C32 - @pytest.mark.skip without reason { #c32 }
`J1` · LOW · F5

`@pytest.mark.skip` with no `reason=`. No explanation for the disabled test; may be forgotten
permanently.

### C35 - retry / flaky decorator { #c35 }
`J6` · LOW · F6

A decorator named `flaky`, `repeat`, `retry`, `rerun`, or `flake` on a test. Masks
non-determinism instead of fixing it.

---

## Family E - it passes, but checks the wrong thing

### C33 - ML metric computed but not asserted { #c33 }
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

### C36 - pytest.fail() without reason { #c36 }
`J1` · LOW · F8

`pytest.fail()` with no message. The failure is unintelligible in CI output.

### C37 - duplicate parametrize case { #c37 }
`J2` · LOW · F8

`@pytest.mark.parametrize` where the same argument set appears twice. The duplicate confirms the
same code path again and adds no coverage.

---

## Family additions (catalog sync)

### C38 - two tests share a name { #c38 }
`J1` · HIGH · F5

Two `def test_*` at module or class scope with the same name. Python binds the later over the
earlier, so the first never runs.

### C39 - returns a comparison instead of asserting { #c39 }
`J1` · HIGH · F1

`return x == y` in a test. pytest ignores the returned value (it warns with
`PytestReturnNotNoneWarning`); nothing is checked.

### C41 - assertion on a None-returning mutator { #c41 }
`J4` · LOW · F3

`assert not lst.sort()` / `assertIsNone(lst.sort())`. Whether it is trivially green depends on
the receiver's type, so this is a skill-only judgment, restricted to known mutators
(`sort`, `append`, `extend`, `reverse`, `update`, `add`, `remove`, `insert`, `clear`).

### C42 - assertion on a generator or lambda { #c42 }
`J2` · HIGH · F3

`assert (x for x in y)` / `assert lambda: ...`. The object is always truthy. A list, set, or
dict comprehension is not C42, because it can be empty.

### C43 - mid-test skip { #c43 }
`J1` · LOW · F5

`pytest.skip()` after test logic, with checks below it that then never run. A skip at the top is
a legitimate guard.

### C44 - numeric tautology { #c44 }
`J2` · HIGH · F3

`len(x) >= 0`, `abs(x) >= 0`, `len(x) > -1`, or a mock's `call_count >= 0` / `> -1`. The
comparison is always true.

### C45 - empty parametrize { #c45 }
`J1` · HIGH · F5

`@pytest.mark.parametrize("...", [])`. Zero cases are generated, so the test never runs.

### C48 - dark patch: flips a test-mode flag then asserts { #c48 }
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

### C59 - bare top-level comparison statement { #c59 }
`J1` · HIGH · F1

`result == expected` written as a statement, not inside an `assert`. Python computes the
comparison and throws the result away, so nothing is checked. The loose-statement sibling of
C39 (which returns the comparison); C59 owns the line so C2b does not double-report.

=== "BAD"
    ```python
    def test_total():
        total(items) == 42       # C59 - computed and discarded, no assert
    ```
=== "CLEAN"
    ```python
    def test_total():
        assert total(items) == 42
    ```

---

## Project layer - config audit (PL)

Emitted by the config-audit pass, not the per-file scan. The suite goes green by configuration,
not by a smell inside any one test file.

### PL1 - assertions stripped at runtime { #pl1 }
`J1` · LOW · F2

`python -O` / `-OO` or `PYTHONOPTIMIZE` set in the run config strips every `assert` at byte-compile
time, so the whole suite passes with no checks. Run without `-O` and unset `PYTHONOPTIMIZE`.

### PL2 - warnings not promoted to errors { #pl2 }
`J1` · LOW · F4

The pytest `filterwarnings` config does not turn warnings into errors, so deprecations and runtime
warnings pass silently. Set `filterwarnings = error` to make them fail the suite.

### PL7 - no coverage gate { #pl7 }
`J5` · LOW · F8

No `--cov-fail-under` / `[tool.coverage.report] fail_under` is configured, so coverage can fall to
zero and the suite still passes. Add a coverage threshold to gate it.

### PL8 - the run stops early { #pl8 }
`J5` · LOW · F5

`addopts` carries `-x` / `--maxfail` / `--exitfirst`, so the run stops on the first failures and the
reported test count is incomplete. Drop them so the full suite runs.

---

## Diagnostic codes (opt-in, OFF by default) { #diagnostics }

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
