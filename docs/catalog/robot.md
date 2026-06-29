# Robot Framework catalog

The codes implemented by [robotframework-falsegreen](../scanners/robot.md): a static scan over the
official Robot Framework parser (`robot.api.get_model`), no execution. A `.robot` file is a DSL,
so this is a model-based pass that maps each finding to [J1-J6](../concepts/judgments.md).

Codes share an id with [Python](python.md) where the concept matches; `R*` codes are
Robot-specific. Confidence: **HIGH** blocks, **LOW** warns, **OFF** is diagnostic-only.

Every emitted code has its own entry below. The index links straight to each one.

## Index

| Code | Conf | J | One-liner |
|---|---|---|---|
| [C2](#c2) | HIGH | J1 | empty test/task/keyword (no keywords run) |
| [C2b](#c2b) | LOW | J1 | runs keywords but no verification keyword |
| [C3](#c3) | HIGH | J1 | swallowed failure (`Run Keyword And Ignore Error`, `TRY/EXCEPT`) |
| [C5](#c5) | HIGH | J2 | always-true (`Should Be True ${TRUE}`) |
| [C6](#c6) | LOW | J4 | `Should Be True` on a bare variable (truthiness only) |
| [C7](#c7) | HIGH | J2 | self-compare (`Should Be Equal ${x} ${x}`) |
| [C9](#c9) | LOW | J4 | catch-all expected error (`Run Keyword And Expect Error *`) |
| [C9b](#c9b) | LOW | J1 | RequestsLibrary `expected_status=any` disables the oracle |
| [C11a](#c11a) | HIGH | J2 | self-confirming literal (a copy of the actual feeds expected) |
| [C16](#c16) | LOW | J1 | `Sleep`, a clock read, or randomness |
| [C20](#c20) | HIGH | J1 | verification after a terminator (`[Return]`/`Fail`/`Pass Execution`) |
| [C21](#c21) | LOW | J1 | verification only inside `IF`/`Run Keyword If` |
| [C23](#c23) | LOW | J6 | hard-coded IP-address URL in test data |
| [C31](#c31) | LOW | J4 | captured value never used (`${x}= Get Text`) |
| [C32](#c32) | LOW | J1 | skipped test (`robot:skip` / `Skip`) |
| [C37](#c37) | LOW | J4 | duplicate `[Template]` data row |
| [C44](#c44) | HIGH | J2 | vacuous library assertion (true for any value) |
| [CC](#cc) | LOW | J1 | commented-out verification keyword |
| [R1](#r1) | HIGH | J1 | `Pass Execution` forces green |
| [R2](#r2) | LOW | J1 | hollow verifier keyword (named like an oracle, asserts nothing) |
| [R3](#r3) | HIGH | J1 | `*** Test Cases ***` in a `.resource` file (never run) |
| [R4](#r4) | HIGH | J1 | `No Operation` is the only step |
| [R5](#r5) | HIGH | J1 | `[Template]` with no data rows |
| [R6](#r6) | LOW | J4 | `Should Be True` on a string literal (always truthy) |
| [R7](#r7) | LOW | J1 | hollow template keyword (every generated case has no oracle) |
| [R8](#r8) | HIGH | J1 | the only verification lives in `[Setup]` |
| [R8b](#r8b) | LOW | J1 | the only verification lives in `[Teardown]` |
| [D2](#diagnostic-codes-opt-in-off-by-default) | OFF | J4 | control flow at the test level |
| [M2](#diagnostic-codes-opt-in-off-by-default) | OFF | J5 | over-long test |
| [PL9](#pl9) | LOW | J1 | skip-on-failure / noncritical turns a fail into a pass |

A code keeps its id across languages where the smell is the same; the signal below is the Robot
form. Two shared ids carry a meaning specific to Robot, both documented here:

- **C31** here is a captured value that is never used (`${x}= Get Text locator` with no later
  assertion on `${x}`). In [Python](python.md#c31) C31 is the `capsys`/`capfd` capture discarded.
  Same concept (a captured value goes unasserted), different mechanism.
- **C44** here is widened beyond the numeric tautology it names in [Python](python.md#c44) and
  [JS](javascript-typescript.md#c44) to any vacuous library assertion (`Should Contain ${EMPTY}`,
  `Should Not Be Empty ${TRUE}`, a `Length Should Be` tautology). Same id, broader bucket,
  documented rather than silent drift.

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

The signal is the Robot form; the failure mode is the same as the Python entry of the same id.

### C2 - empty test, task, or keyword { #c2 }
`J1` · HIGH · F1

An empty test case (only settings, no body keywords), task, or user keyword. Nothing runs.

### C2b - runs keywords but no verification keyword { #c2b }
`J1` · LOW · F1

The body runs keywords but none of them verifies anything (no `Should`, no library assertion). No
oracle.

### C3 - swallowed failure { #c3 }
`J1` · HIGH · F2

`Run Keyword And Ignore Error` / `Run Keyword And Return Status` whose status is never asserted, or
a `TRY/EXCEPT` that swallows the failure. The status goes unchecked.

### C5 - always-true check { #c5 }
`J2` · HIGH · F3

`Should Be True    ${TRUE}`, `Should Be Equal` with two equal literals, or a constant-true
`Set Variable If` feeding the expected side. Always passes.

### C6 - weak check on a bare variable { #c6 }
`J4` · LOW · F4

`Should Be True    ${x}` on a bare variable checks truthiness only, not a comparison. Compare the
value with `Should Be Equal`.

=== "BAD"
    ```robotframework
    Login Works
        ${ok}=    Login
        Should Be True    ${ok}       # C6 - only truthiness
    ```
=== "CLEAN"
    ```robotframework
    Login Works
        ${ok}=    Login
        Should Be Equal    ${ok}    ${TRUE}
    ```

### C7 - self-compare { #c7 }
`J2` · HIGH · F3

`Should Be Equal    ${x}    ${x}`: both sides are the same variable, always equal.

### C9 - catch-all expected error { #c9 }
`J4` · LOW · F4

`Run Keyword And Expect Error    *` (or `GLOB:*` / `REGEXP:.*`) accepts any error, including one
the test never meant to trigger.

### C9b - `expected_status=any` disables the oracle { #c9b }
`J1` · LOW · F4

A RequestsLibrary HTTP method with `expected_status=any` / `anything`: the request accepts every
status, so the oracle is disabled and a 500 never fails.

### C11a - self-confirming literal { #c11a }
`J2` · HIGH · F3

An in-body copy of the actual feeds the expected side: `${y}=    Set Variable    ${x}`, then
`Should Be Equal    ${x}    ${y}`. The oracle confirms itself.

### C16 - non-deterministic source { #c16 }
`J1` · LOW · F6

`Sleep` used as synchronization instead of `Wait Until *`, `Get Current Date` (clock read),
`Generate Random String` (randomness), or `Evaluate` with `datetime`/`random`/`uuid`.

### C20 - verification after a terminator { #c20 }
`J1` · HIGH · F2

A verification keyword after `[Return]`, `Return From Keyword`, `Fail`, or `Pass Execution` in the
same block: a dead step that never runs.

### C21 - verification only runs conditionally { #c21 }
`J1` · LOW · F2

The only verification sits inside an `IF` / `Run Keyword If` that may not run, so the test can pass
without checking anything.

### C23 - hard-coded IP-address URL { #c23 }
`J6` · LOW · F6

A hard-coded IP-address URL in test data: environment coupling / mystery guest. Read it from a
variable or resource.

### C31 - captured value never used { #c31 }
`J4` · LOW · F1

`${x}=    Get Text    locator` (or a similar capture) whose result is never asserted later. The
capture is dead; the test verifies something else. Robot's analogue of the Python `capsys` C31,
different mechanism, same idea: a captured value goes unasserted.

=== "BAD"
    ```robotframework
    Title Is Set
        ${title}=    Get Text    h1     # C31 - captured, never asserted
        Click    submit
    ```
=== "CLEAN"
    ```robotframework
    Title Is Set
        ${title}=    Get Text    h1
        Should Be Equal    ${title}    Welcome
    ```

### C32 - skipped test { #c32 }
`J1` · LOW · F5

`[Tags]    robot:skip` or the `Skip` keyword leaves the test out of the run.

### C37 - duplicate `[Template]` data row { #c37 }
`J4` · LOW · F8

The same data row appears twice in a `[Template]`: the duplicate runs the same scenario and adds no
coverage.

### C44 - vacuous library assertion { #c44 }
`J2` · HIGH · F3

A library assertion provably true for any value: `Should Contain    ${x}    ${EMPTY}`,
`Should Not Be Empty    ${TRUE}`, `Should Be Empty    ${EMPTY}`, a `Length Should Be` tautology.
Widened from the numeric form the same id names in Python/JS.

### CC - commented-out verification keyword { #cc }
`J1` · LOW · F2

A line in the body is a commented-out verification keyword (`# Should Be Equal ...`): the oracle is
switched off.

## Robot-specific codes

### R1 - forced green { #r1 }
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

### R2 - hollow verifier keyword { #r2 }
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

### R3 - test cases in a .resource file { #r3 }
`J1` · HIGH · F5

A `*** Test Cases ***` section in a `.resource` file is invalid; the cases never run.

### R4 - No Operation only { #r4 }
`J1` · HIGH · F1

The only step is `No Operation` - the test runs but does nothing.

### R5 - empty [Template] { #r5 }
`J1` · HIGH · F5

A `[Template]` keyword with no data rows generates zero cases. Parallel of `C45` / `JS22`.

### R6 - Should Be True on a string literal { #r6 }
`J4` · HIGH · F3

`Should Be True    some text` passes a non-empty string, which is always truthy, so the check
never fails. Pass a real expression (`${x} > 0`).

### R7 - hollow template keyword { #r7 }
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

### R8 - the only verification lives in [Setup] { #r8 }
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

### R8b - the only verification lives in [Teardown] { #r8b }
`J1` · LOW · F2

The sole verification keyword sits in `[Teardown]` (or `Test Teardown`). Teardown runs even when
the body fails and reports on a separate axis (it can flip a failed test's reason), so it does not
verify the body's result. Lower confidence than R8 because a teardown check is sometimes a
deliberate cleanup assertion.

## Project layer - run config (PL)

### PL9 - skip-on-failure turns a fail into a pass { #pl9 }
`J1` · LOW · F5

A `skip-on-failure` / `noncritical` setting in the run config turns a failing test into a non-fatal
pass. Legacy (removed in Robot Framework 4+), still seen in older suites.

## Diagnostic codes (opt-in, OFF by default)

Family F8: hygiene, not false-green. Robocop also covers these.

| Code | What it flags |
|---|---|
| **D2** | control flow (`IF`/`FOR`/`WHILE`/`TRY`) at the test/task level |
| **M2** | over-long test/task (the guide suggests max ~10 steps) |

## Look-alikes: do NOT flag

- `Run Keyword And Expect Error` with a SPECIFIC message/pattern: it is asserting.
- `Wait Until Keyword Succeeds`: a legitimate retry for E2E flakiness, not a Sleep smell.
- Teardown keywords (`[Teardown]`, `Close Browser`): cleanup, not the oracle.
- E2E presence keywords (`Page Should Contain Element`): the assertion at the browser layer.
- `Run Keywords    Click    AND    Should Be Equal ...`: the AND chain contains a real check; not
  C2b.
