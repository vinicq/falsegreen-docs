# Robot Framework catalog

The codes implemented by [robotframework-falsegreen](../scanners/robot.md): a static scan over the
official Robot Framework parser (`robot.api.get_model`), no execution. A `.robot` file is a DSL,
so this is a model-based pass that maps each finding to [J1-J6](../concepts/judgments.md).

Codes share an id with [Python](python.md) where the concept matches; `R*` codes are
Robot-specific.

## What counts as verification (the oracle)

The whole false-green check hinges on recognizing the assertion keywords, so a real check is not
mistaken for "no verification". The dominant convention is the word **`Should`**, plus
library-specific forms:

- **BuiltIn / Collections / String:** `Should Be Equal`, `Should Be True`, `Should Contain`,
  `Should Match`, `Length Should Be`, `List Should Contain Value`, `Dictionary Should Contain Key`.
- **SeleniumLibrary / AppiumLibrary:** `Page Should Contain*`, `Element Should Be Visible`,
  `Element Text Should Be`, `Title Should Be`. `Wait Until Page Contains` / `Wait Until Element Is
  Visible` also verify (they fail on timeout).
- **Browser (Playwright):** the assertion engine `Get ...    <selector>    <operator>    <expected>`
  where operator is `==`, `!=`, `contains`, etc. A bare `Get Text    h1` with no operator verifies
  nothing.
- **RequestsLibrary:** `Status Should Be`, `Request Should Be Successful`, and a request keyword
  (`GET`/`POST`/...) carrying `expected_status=<code>` (it fails if the status differs).
  `expected_status=any` disables the check, so it does not count.
- **RESTinstance:** schema keywords (`Integer`, `Number`, `String`, `Object`, ...).
- **DatabaseLibrary:** `Row Count Should Be Equal`, `Check If (Not) Exists In Database`.
- A project **custom keyword** whose name contains `Should`/`Verify`/`Assert`/`Check` and whose
  body actually calls one of the above.

A test with none of these (only `Click`, `Go To`, `Input Text`, `Log`, a bare `Get *`) verifies
nothing.

## Shared codes (same concept as Python)

| Code | Conf | Robot form |
|---|---|---|
| C2 | HIGH | empty test case (only settings, no body keywords) / empty user keyword |
| C2b | LOW | runs keywords but no verification keyword |
| C3 | HIGH | swallowed failure: `Run Keyword And Ignore Error` / `Return Status` not asserted, or a `TRY/EXCEPT` that swallows |
| C5 | HIGH | always-true (`Should Be True    ${TRUE}`, or a constant-true `Set Variable If` feeding the expected side of `Should Be Equal`) |
| C7 | HIGH | self-compare (`Should Be Equal    ${x}    ${x}`) |
| C44 | HIGH | library assertion provably true for any value (`Should Contain    ${x}    ${EMPTY}`, `Should Not Be Empty    ${TRUE}`, `Should Be Empty    ${EMPTY}`, a `Length Should Be` tautology) |
| C9 | LOW | catch-all expected error (`Run Keyword And Expect Error    *`) |
| C9b | LOW | a RequestsLibrary HTTP method with `expected_status=any` / `anything` — the request accepts every status, so the oracle is disabled and a 500 never fails |
| C11a | HIGH | self-confirming literal: an in-body copy of the actual feeds the expected side (`${y}=    Set Variable    ${x}`, then `Should Be Equal    ${x}    ${y}`) |
| C16 | LOW | `Sleep` as synchronization instead of `Wait Until *`, `Get Current Date` (clock read), `Generate Random String` (randomness), or `Evaluate` with `datetime`/`random`/`uuid` |
| C20 | HIGH | verification after a terminator (`[Return]`, `Return From Keyword`, `Fail`, `Pass Execution`) |
| C21 | LOW | verification only inside `IF` / `Run Keyword If` that may not run |
| C23 | LOW | hard-coded IP-address URL in test data |
| C32 | LOW | skipped (`[Tags]    robot:skip` / `Skip`) |
| C37 | LOW | duplicate `[Template]` data row |
| CC | LOW | commented-out verification keyword |

## Robot-specific codes

### R1 - forced green
`J1` · HIGH · F3

`Pass Execution` (or `Pass Execution If` with an always-true condition) forces the test to pass
regardless of any check.

=== "BAD"
    ```robotframework
    Login Works
        Open App
        Pass Execution    skipping for now
    ```
=== "CLEAN"
    ```robotframework
    Login Works
        Open App
        Page Should Contain    Welcome
    ```

### R2 - hollow verifier keyword
`J1` · LOW · F1

A user keyword named like an oracle (`Verify *`, `Assert *`, `Should *`, `Check *`) whose body
contains no verification keyword. A test calling `Verify Login` looks protected but asserts
nothing - the root cause of a missed C2b.

=== "BAD"
    ```robotframework
    *** Keywords ***
    Verify Login
        Log    logged in       # no Should/assertion - hollow
    ```
=== "CLEAN"
    ```robotframework
    *** Keywords ***
    Verify Login
        Page Should Contain    Welcome
    ```

### R3 - test cases in a .resource file
`J1` · HIGH · F5

A `*** Test Cases ***` section in a `.resource` file is invalid; the cases never run.

### R4 - No Operation only
`J1` · HIGH · F1

The only step is `No Operation` - the test runs but does nothing.

### R5 - empty [Template]
`J1` · HIGH · F5

A `[Template]` keyword with no data rows generates zero cases. Parallel of `C45` / `JS22`.

### R6 - Should Be True on a string literal
`J4` · HIGH · F3

`Should Be True    some text` passes a non-empty string, which is always truthy, so the check
never fails. Pass a real expression (`${x} > 0`).

### R7 - hollow template keyword
`J1` · LOW · F1

A `[Template]` test whose template keyword is defined in the same file and contains no
verification: every data row passes for free. Only flagged when the template keyword resolves
in-file; an external/imported template keyword is left alone (it may verify via a keyword the
scanner cannot see).

=== "BAD"
    ```robotframework
    *** Test Cases ***
    Check Logins
        [Template]    Do Login
        alice    secret
        bob      hunter2

    *** Keywords ***
    Do Login
        [Arguments]    ${user}    ${pass}
        Input Text    user    ${user}
        Click    submit          # no verification - every row is green
    ```
=== "CLEAN"
    ```robotframework
    *** Keywords ***
    Do Login
        [Arguments]    ${user}    ${pass}
        Input Text    user    ${user}
        Click    submit
        Page Should Contain    Welcome
    ```

### R8 - the only verification lives in [Setup]
`J1` · HIGH · F2

A test whose sole verification keyword sits in `[Setup]` (or a suite-level `Test Setup`). Setup
runs before the body acts, so it checks preconditions, not the result: the body can break and the
suite stays green. Move the assertion into the body.

=== "BAD"
    ```robotframework
    Order Is Shipped
        [Setup]    Should Be Equal    ${status}    pending
        Ship Order                      # body acts, never verified
    ```
=== "CLEAN"
    ```robotframework
    Order Is Shipped
        Ship Order
        Should Be Equal    ${order.status}    shipped
    ```

### R8b - the only verification lives in [Teardown]
`J1` · LOW · F2

The sole verification keyword sits in `[Teardown]` (or `Test Teardown`). Teardown runs even when
the body fails and reports on a separate axis (it can flip a failed test's reason), so it does not
verify the body's result. Lower confidence than R8 because a teardown check is sometimes a
deliberate cleanup assertion.

## Diagnostic codes (opt-in, OFF by default)

`D2` (control flow in a test) and `M2` (over-long test) - hygiene, not false-green. Robocop also
covers these.

## Look-alikes: do NOT flag

- `Run Keyword And Expect Error` with a SPECIFIC message/pattern: it is asserting.
- `Wait Until Keyword Succeeds`: a legitimate retry for E2E flakiness, not a Sleep smell.
- Teardown keywords (`[Teardown]`, `Close Browser`): cleanup, not the oracle.
- E2E presence keywords (`Page Should Contain Element`): the assertion at the browser layer.
- `Run Keywords    Click    AND    Should Be Equal ...`: the AND chain contains a real check; not
  C2b.
