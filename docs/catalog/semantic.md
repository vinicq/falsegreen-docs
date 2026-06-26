# Semantic catalog (LLM)

The patterns only the [skill](../scanners/skill.md) can catch. They need reading the test against
its intent, the spec, and the production code: no parser or linter sees them. This is the F7
layer, and the reason the skill exists on top of the static scanners.

Confidence on these is operator-confirmed: treat as LOW or HIGH by how clear the contradiction
is. Never auto-block without showing the reasoning, and never report a wrong-value finding without
citing an independent [oracle](../concepts/oracle.md).

## The case catalog

These five cases are the canonical semantic false-greens, detected by reading the test.

### Case 10 - mocks the unit under test
`J3` · HIGH

The test patches or mocks the function it is supposed to test, then asserts on the mock's return
value. It tests the mock configuration, not the code.

=== "BAD"
    ```python
    @patch('mymodule.add')
    def test_add(mock_add):
        mock_add.return_value = 5
        assert add(2, 3) == 5          # asserts the mock's value
    ```
=== "CLEAN"
    ```python
    @patch('mymodule.db.fetch')        # mock the edge, not the unit
    def test_get_user(mock_fetch):
        mock_fetch.return_value = {'id': 1, 'name': 'Alice'}
        user = get_user(1)
        assert user.name == 'Alice'
    ```

### Case 11 - asserts the value fed to the mock
`J2` / `J3` · HIGH

The test stubs a dependency to return X, then asserts the result equals X. The result passes
through no production logic - it is an echo.

=== "BAD"
    ```python
    def test_price(mock_product):
        mock_product.price = 100
        assert get_price(mock_product) == 100   # echoes the stub
    ```
=== "CLEAN"
    ```python
    def test_price_with_tax(mock_product):
        mock_product.price = 100
        assert get_price_with_tax(mock_product) == 110   # real logic
    ```

### Case 12 - re-implements the production formula
`J2` · HIGH

The expected value is computed with the same formula as the code. Both agree on the same wrong
answer.

=== "BAD"
    ```python
    def test_total():
        expected = price + price * tax_rate   # re-implements the SUT
        assert calculate_total(price, tax_rate) == expected
    ```
=== "CLEAN"
    ```python
    def test_total():
        assert calculate_total(100, 0.1) == 110.0   # from the spec
    ```

### Case 15 - passes only if another test ran first
`J6` · HIGH

The test reads shared mutable state a sibling set up. It passes in a specific order and fails when
run alone.

### Case 18 - expected value contradicts what the code should do
`J2` · HIGH

The test asserts a value that contradicts the spec, freezing a bug as correct. **Requires an
independent oracle cited before reporting.**

## The S-series (AI-only)

Patterns no AST or linter sees. Each maps to a judgment.

| Code | J | What it detects |
|---|---|---|
| **S1** | J4 | intent mismatch: the name claims to verify X, the assertion checks Y or a trivial property |
| **S2** | J4 | irrelevant oracle: asserts a property unrelated to the behavior under test |
| **S3** | J2 | plausible-but-wrong expected value (off-by-one, wrong rounding); deeper than case 18 |
| **S4** | J4 | the oracle cannot tell correct from a likely bug (`len(result) == 3` when the bug also yields three) |
| **S5** | J3 | tests the framework, not the code (a dict stores a key, the ORM returns what was saved) |
| **S6** | J4 | happy-path only against a stated contract that promises error handling |
| **S7** | J2 | expected value lifted from the output (a pasted dict, a captured response) |
| **S8** | J3 | the mock's return reaches the assertion through an indirection |
| **S9** | J2 | self-fulfilling arrangement: arranges the exact state it then asserts |
| **S10** | J4 | asserts the log, not the effect the message describes |
| **S11** | J4 | negative-only assertion on a security filter (`secret not in output`) with no paired positive |
| **S12** | J3 | patches core logic instead of an external edge (deeper than case 10) |
| **S13** | J6 | passes only via shared state a sibling set up, across files the AST cannot prove |

## Look-alikes: do NOT flag

- A deliberately narrow unit test whose scope the spec confirms (S6 needs a stated broader
  contract).
- A constant the spec genuinely endorses (not S3).
- A sanitizer test that already pairs the negative check with a positive one (not S11).
- A mock on a genuine external edge: DB, network, clock (not S12).
- A test whose shared state is reset by an autouse / `beforeEach` teardown (not S13).
