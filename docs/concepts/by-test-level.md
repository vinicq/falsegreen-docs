# Patterns by language and test level

This page lists what each scanner in the family detects, organized two ways: by language (a
fixed axis) and by test level (unit, integration, E2E). The catalog pages carry the full
definitions; this is the map across them.

## An honest note about level

Unit, integration, and E2E are **not a fixed partition by code**. The level is a per-finding
classification, decided at [J3](judgments.md): does the test exercise the real unit, a
collaborator or integration boundary, or the full E2E stack. The same false-green pattern reads
at whichever level the test operates. So the list below is complete per language (the fixed
axis), and the level clusters at the end are the patterns *characteristic* of each level, with
the caveat that most codes can show up at more than one level.

## The model

The judgments [J1-J6](judgments.md) ask, in order: does the assertion run, is the oracle
independent, does it exercise the real unit or a double, does it verify enough, is it coupled to
internals, does it pass in isolation. The [skill](../scanners/skill.md) is the superset of the
three structural scanners: the structural codes (`C*` / `JS*` / `R*` / `PL*`) plus the semantic
ones (`S1`-`S21`), which need an intent read that an AST cannot decide.

## Python (falsegreen)

The [falsegreen](../scanners/python.md) scanner emits 67 codes over pytest and unittest. Each
links to its [catalog entry](../catalog/python.md).

| Code | J | Conf | What it catches |
|---|---|---|---|
| [C1](../catalog/python.md#c1) | J1 | LOW | assertion inside a conditional or loop that may never run |
| [C2](../catalog/python.md#c2) | J1 | HIGH | test body has no assertion at all |
| [C3](../catalog/python.md#c3) | J1 | HIGH | assert inside a try whose except swallows the error |
| [C4](../catalog/python.md#c4) | J1 | HIGH | test function not collected by pytest |
| [C5](../catalog/python.md#c5) | J2 | HIGH | always-true assertion |
| [C6](../catalog/python.md#c6) | J4 | LOW | weak assertion: only checks something came back |
| [C7](../catalog/python.md#c7) | J2 | HIGH | self-comparison: both sides are identical |
| [C8](../catalog/python.md#c8) | J4 | LOW | float exact equality |
| [C9](../catalog/python.md#c9) | J4 | LOW | `pytest.raises` too broad |
| [CC](../catalog/python.md#cc) | J1 | LOW | commented-out assert |
| [C13](../catalog/python.md#c13) | J4 | HIGH | mock assertion misspelled or not called |
| [C14](../catalog/python.md#c14) | J2 | LOW | golden file generated from the actual output |
| [C16](../catalog/python.md#c16) | J6 | LOW | result depends on uncontrolled time, randomness, or sleep |
| [C17](../catalog/python.md#c17) | J1 | HIGH | `pytest.skip()` inside a broad except |
| [C18](../catalog/python.md#c18) | J2 | LOW | string/repr comparison |
| [C19](../catalog/python.md#c19) | J1 | LOW | `pytest.raises` wraps more than one call |
| [C20](../catalog/python.md#c20) | J1 | HIGH | assertion after an unconditional return/raise/fail |
| [C21](../catalog/python.md#c21) | J1 | LOW | every assertion is inside a conditional; none runs unconditionally |
| [C22](../catalog/python.md#c22) | J1 | OFF | async test never awaits the unit under test |
| [C23](../catalog/python.md#c23) | J6 | LOW | hard-coded absolute or home-relative file path |
| [C24](../catalog/python.md#c24) | J6 | LOW | module-level mutable state mutated by the test |
| [C25](../catalog/python.md#c25) | J1 | LOW | xfail without `strict=True` |
| [C27](../catalog/python.md#c27) | J1 | HIGH | try/except/pass around the SUT call with no assertion |
| [C28](../catalog/python.md#c28) | J4 | LOW | `pytest.raises` binding variable never read |
| [C29](../catalog/python.md#c29) | J6 | LOW | `os.environ` modified directly in the test |
| [C2b](../catalog/python.md#c2b) | J1 | LOW | calls production code but verifies nothing |
| [C2c](../catalog/python.md#c2c) | J1 | LOW | empty `self.subTest(...)` block |
| [C30](../catalog/python.md#c30) | J3 | LOW | HTTP mock not activated |
| [C31](../catalog/python.md#c31) | J4 | LOW | `capsys.readouterr()` result discarded |
| [C32](../catalog/python.md#c32) | J1 | LOW | `@pytest.mark.skip` without a reason |
| [C33](../catalog/python.md#c33) | J4 | LOW | ML metric computed but not asserted |
| [C34](../catalog/python.md#c34) | J4 | LOW | suboptimal assertion form |
| [C35](../catalog/python.md#c35) | J6 | LOW | retry/flaky decorator |
| [C36](../catalog/python.md#c36) | J1 | LOW | `pytest.fail()` without a reason |
| [C37](../catalog/python.md#c37) | J2 | LOW | duplicate parametrize case |
| [C38](../catalog/python.md#c38) | J1 | HIGH | two tests share a name |
| [C39](../catalog/python.md#c39) | J1 | HIGH | returns a comparison instead of asserting |
| [C41](../catalog/python.md#c41) | J4 | LOW | assertion on a None-returning mutator |
| [C42](../catalog/python.md#c42) | J2 | HIGH | assertion on a generator or lambda |
| [C43](../catalog/python.md#c43) | J1 | LOW | mid-test skip |
| [C44](../catalog/python.md#c44) | J2 | HIGH | numeric tautology |
| [C45](../catalog/python.md#c45) | J1 | HIGH | empty parametrize |
| [C48](../catalog/python.md#c48) | J1 | LOW | dark patch: flips a test-mode flag then asserts |
| [C49](../catalog/python.md#c49) | J1 | LOW | `pytest.warns`/`assertWarns` wraps more than one call |
| [C4b](../catalog/python.md#c4b) | J1 | LOW | test class has `__init__` (pytest will not collect it) |
| [C50](../catalog/python.md#c50) | J4 | LOW | captured log never asserted |
| [C51](../catalog/python.md#c51) | J1 | HIGH | empty-bodied `pytest.raises`/`warns` context |
| [C52](../catalog/python.md#c52) | J2 | LOW | membership self-confirmation |
| [C55](../catalog/python.md#c55) | J3 | LOW | assertion compares two mock-rooted values |
| [C56](../catalog/python.md#c56) | J1 | LOW | sync assert of a never-awaited coroutine |
| [C57](../catalog/python.md#c57) | J3 | LOW | assertion against an unconfigured Mock attribute |
| [C59](../catalog/python.md#c59) | J1 | HIGH | bare comparison written as a statement |
| [C6b](../catalog/python.md#c6b) | J3 | LOW | assertion on a positional mock argument via computed index |
| [C6c](../catalog/python.md#c6c) | J4 | LOW | mock `call_count` truthiness as the oracle |
| [C8b](../catalog/python.md#c8b) | J4 | LOW | approximate equality with no explicit tolerance |
| [C11a](../catalog/python.md#c11a) | J2 | LOW | self-confirming literal: test assigns then asserts the same value |
| [C13b](../catalog/python.md#c13b) | J3 | LOW | `patch()` without autospec |
| [D1](../catalog/python.md#diagnostics) | - | LOW | assertion roulette: multiple asserts, none with a message |
| [D3](../catalog/python.md#diagnostics) | - | LOW | duplicate assert: same assertion appears twice |
| [D4](../catalog/python.md#diagnostics) | - | LOW | unnamed parametrize cases |
| [D5](../catalog/python.md#diagnostics) | - | LOW | excessive inline setup |
| [D6](../catalog/python.md#diagnostics) | - | LOW | debug print in the test |
| [M2](../catalog/python.md#diagnostics) | - | LOW | long test method |
| [PL1](../catalog/python.md#pl1) | J1 | n/a | asserts stripped at runtime |
| [PL2](../catalog/python.md#pl2) | J1 | n/a | warnings not promoted |
| [PL7](../catalog/python.md#pl7) | J5 | n/a | no coverage gate |
| [PL8](../catalog/python.md#pl8) | J5 | n/a | run stops early |

## JavaScript / TypeScript and the TSX/JSX family (falsegreen-js)

The [falsegreen-js](../scanners/javascript.md) scanner emits 52 codes: the JS-specific ones plus
the shared `C*`. Each links to its [catalog entry](../catalog/javascript-typescript.md).

| Code | J | Conf | What it catches |
|---|---|---|---|
| [C2](../catalog/javascript-typescript.md#c2) | J1 | HIGH | test body has no assertion at all |
| [C5](../catalog/javascript-typescript.md#c5) | J2 | HIGH | always-true assertion |
| [C6](../catalog/javascript-typescript.md#c6) | J4 | LOW | weak assertion: only checks something came back |
| [C7](../catalog/javascript-typescript.md#c7) | J2 | HIGH | self-comparison: both sides are identical |
| [C8](../catalog/javascript-typescript.md#c8) | J4 | LOW | float exact equality |
| [C9](../catalog/javascript-typescript.md#c9) | J4 | LOW | exception matcher too broad |
| [CC](../catalog/javascript-typescript.md#cc) | J1 | LOW | commented-out assert |
| [C16](../catalog/javascript-typescript.md#c16) | J6 | LOW | result depends on uncontrolled time, randomness, or sleep |
| [C18](../catalog/javascript-typescript.md#c18) | J2 | LOW | string/repr comparison |
| [C20](../catalog/javascript-typescript.md#c20) | J1 | HIGH | assertion after an unconditional return/raise/fail |
| [C21](../catalog/javascript-typescript.md#c21) | J1 | LOW | every assertion is inside a conditional; none runs unconditionally |
| [C23](../catalog/javascript-typescript.md#c23) | J6 | LOW | hard-coded absolute or home-relative file path |
| [C2b](../catalog/javascript-typescript.md#c2b) | J1 | LOW | calls production code but verifies nothing |
| [C37](../catalog/javascript-typescript.md#c37) | J2 | LOW | duplicate parametrize case |
| [C44](../catalog/javascript-typescript.md#c44) | J2 | HIGH | numeric tautology |
| [C48](../catalog/javascript-typescript.md#c48) | J1 | LOW | dark patch: flips a test-mode flag then asserts |
| [C8b](../catalog/javascript-typescript.md#c8b) | J4 | LOW | approximate equality with no explicit tolerance |
| [C11a](../catalog/javascript-typescript.md#c11a) | J2 | LOW | self-confirming literal: test assigns then asserts the same value |
| [D1](../catalog/javascript-typescript.md#diagnostics) | - | LOW | assertion roulette: multiple asserts, none with a message |
| [D3](../catalog/javascript-typescript.md#diagnostics) | - | LOW | duplicate assert: same assertion appears twice |
| [D4](../catalog/javascript-typescript.md#diagnostics) | - | LOW | unnamed parametrize cases |
| [D6](../catalog/javascript-typescript.md#diagnostics) | - | LOW | debug print in the test |
| [D7](../catalog/javascript-typescript.md#diagnostics) | - | LOW | anonymous test: empty or missing description |
| [D8](../catalog/javascript-typescript.md#diagnostics) | - | LOW | magic number in an assertion |
| [JS1](../catalog/javascript-typescript.md#js1) | - | HIGH | focused test (`it.only`/`fit`) skips the rest of the suite |
| [JS2](../catalog/javascript-typescript.md#js2) | - | HIGH | `expect(x)` with no matcher |
| [JS3](../catalog/javascript-typescript.md#js3) | - | LOW | snapshot is the only assertion |
| [JS4](../catalog/javascript-typescript.md#js4) | - | LOW | skipped test (`it.skip`/`xit`/`it.todo`) |
| [JS5](../catalog/javascript-typescript.md#js5) | - | LOW | async query/event not awaited (`findBy*`/`waitFor`/user-event) |
| [JS6](../catalog/javascript-typescript.md#js6) | - | HIGH | empty `describe`/`suite` |
| [JS7](../catalog/javascript-typescript.md#js7) | - | LOW | assertion in a non-awaited `setTimeout`/`then` callback |
| [JS8](../catalog/javascript-typescript.md#js8) | - | LOW | mocks the unit under test and asserts it directly |
| [JS9](../catalog/javascript-typescript.md#js9) | - | HIGH | assertion in a dead literal branch (`if(false)`) |
| [JS11](../catalog/javascript-typescript.md#js11) | - | LOW | `try/catch` swallows the assertion |
| [JS13](../catalog/javascript-typescript.md#js13) | - | LOW | `queryBy*` query (returns null when absent) as a loose statement, never asserted; `getBy*`/`findBy*` throw on absence and *are* the assertion |
| [JS15](../catalog/javascript-typescript.md#js15) | - | LOW | comparison wrapped in a boolean (`expect(a===b).toBe(true)`) |
| [JS17](../catalog/javascript-typescript.md#js17) | - | LOW | commented-out test block (`// it(...)`) |
| [JS18](../catalog/javascript-typescript.md#js18) | - | LOW | `done` callback instead of async/await |
| [JS21](../catalog/javascript-typescript.md#js21) | - | HIGH | matcher referenced but never called (`expect(x).toBe` with no `()`) |
| [JS22](../catalog/javascript-typescript.md#js22) | - | HIGH | empty `it.each`/`test.each` table |
| [JS23](../catalog/javascript-typescript.md#js23) | - | HIGH | `expect.assertions(N)` with fewer unconditional reachable `expect()` calls than `N` |
| [JS24](../catalog/javascript-typescript.md#js24) | - | LOW | Cypress `cy.get/find/contains` query with no `.should`/`.and`/`.then` assertion |
| [JS25](../catalog/javascript-typescript.md#js25) | - | HIGH | the only assertion sits inside an array-iterator callback; runs zero times on an empty collection |
| [JS26](../catalog/javascript-typescript.md#js26) | - | LOW | fake timers installed but never advanced; the scheduled callback never fires |
| [JS27](../catalog/javascript-typescript.md#js27) | - | LOW | `toHaveBeenCalled*` is the sole oracle on a locally-created double; verifies wiring, not behaviour |
| [JS29](../catalog/javascript-typescript.md#js29) | - | LOW | `expect(...).resolves`/`.rejects` chain is a bare statement, not awaited or returned |
| [JS30](../catalog/javascript-typescript.md#js30) | - | HIGH | literal-vs-literal assertion (`expect(2).toBe(3)`); both operands fixed at parse time |
| [JS31](../catalog/javascript-typescript.md#js31) | - | LOW | `try/catch` swallows a possible throw with no assertion on the exception |
| [M2](../catalog/javascript-typescript.md#diagnostics) | - | LOW | long test method |
| [PL7](../catalog/javascript-typescript.md#pl7) | J5 | n/a | no coverage gate |
| [PL8](../catalog/javascript-typescript.md#pl8) | J5 | n/a | run stops early |
| [PL10](../catalog/javascript-typescript.md#pl10) | J1 | n/a | `passWithNoTests` |

## Robot Framework (robotframework-falsegreen)

The [robotframework-falsegreen](../scanners/robot.md) scanner emits 30 codes: the `R*`-specific
ones plus the shared `C*`. Each links to its [catalog entry](../catalog/robot.md).

| Code | J | Conf | What it catches |
|---|---|---|---|
| [C2](../catalog/robot.md#c2) | J1 | HIGH | test body has no assertion at all |
| [C3](../catalog/robot.md#c3) | J1 | HIGH | assert inside a try whose except swallows the error |
| [C5](../catalog/robot.md#c5) | J2 | HIGH | always-true assertion |
| [C6](../catalog/robot.md#c6) | J4 | LOW | weak assertion: only checks something came back |
| [C7](../catalog/robot.md#c7) | J2 | HIGH | self-comparison: both sides are identical |
| [C9](../catalog/robot.md#c9) | J4 | LOW | exception matcher too broad |
| [CC](../catalog/robot.md#cc) | J1 | LOW | commented-out assert |
| [C16](../catalog/robot.md#c16) | J6 | LOW | result depends on uncontrolled time, randomness, or sleep |
| [C20](../catalog/robot.md#c20) | J1 | HIGH | assertion after an unconditional return/raise/fail |
| [C21](../catalog/robot.md#c21) | J1 | LOW | every assertion is inside a conditional; none runs unconditionally |
| [C23](../catalog/robot.md#c23) | J6 | LOW | hard-coded absolute or home-relative file path |
| [C2b](../catalog/robot.md#c2b) | J1 | LOW | calls production code but verifies nothing |
| [C31](../catalog/robot.md#c31) | J4 | LOW | capture result discarded |
| [C32](../catalog/robot.md#c32) | J1 | LOW | skip without a reason |
| [C37](../catalog/robot.md#c37) | J2 | LOW | duplicate parametrize case |
| [C44](../catalog/robot.md#c44) | J2 | HIGH | numeric tautology |
| [C9b](../catalog/robot.md#c9b) | J4 | n/a | RequestsLibrary `expected_status=any` |
| [C11a](../catalog/robot.md#c11a) | J2 | LOW | self-confirming literal: test assigns then asserts the same value |
| [D2](../catalog/robot.md#diagnostics) | J4 | n/a | control flow at test level |
| [M2](../catalog/robot.md#diagnostics) | - | LOW | long test method |
| [PL9](../catalog/robot.md#pl9) | J1 | n/a | skip-on-failure run option |
| [R1](../catalog/robot.md#r1) | J1 | n/a | forced green |
| [R2](../catalog/robot.md#r2) | J1 | n/a | hollow verifier keyword |
| [R3](../catalog/robot.md#r3) | J1 | n/a | test cases in a `.resource` |
| [R4](../catalog/robot.md#r4) | J1 | n/a | `No Operation` only |
| [R5](../catalog/robot.md#r5) | J1 | n/a | empty `[Template]` |
| [R6](../catalog/robot.md#r6) | J4 | n/a | `Should Be True` on a string literal |
| [R7](../catalog/robot.md#r7) | J1 | n/a | hollow `[Template]` keyword |
| [R8](../catalog/robot.md#r8) | J4 | n/a | verification only in Setup |
| [R8b](../catalog/robot.md#r8b) | J4 | n/a | verification only in Teardown |

## Semantic codes, skill-only (all languages)

Only the LLM [skill](../scanners/skill.md) detects these. They sit at J2/J3/J4 and need an
intent read that an AST does not decide. Each links to the
[semantic catalog](../catalog/semantic.md).

| Code | J | What it catches |
|---|---|---|
| [S1](../catalog/semantic.md#s-series) | J4 | intent mismatch |
| [S2](../catalog/semantic.md#s-series) | J4 | irrelevant oracle |
| [S3](../catalog/semantic.md#s-series) | J2 | plausible-but-wrong expected value |
| [S4](../catalog/semantic.md#s-series) | J4 | oracle cannot distinguish correct from a likely bug |
| [S5](../catalog/semantic.md#s-series) | J3 | tests the framework, not the code |
| [S6](../catalog/semantic.md#s-series) | J4 | happy-path only against a stated contract |
| [S7](../catalog/semantic.md#s-series) | J2 | expected lifted from the output |
| [S8](../catalog/semantic.md#s-series) | J3 | mock return reaches the assertion through an indirection |
| [S9](../catalog/semantic.md#s-series) | J2 | self-fulfilling arrangement |
| [S10](../catalog/semantic.md#s-series) | J4 | asserts the log, not the effect |
| [S11](../catalog/semantic.md#s-series) | J4 | negative-only assertion on a security filter |
| [S12](../catalog/semantic.md#s-series) | J3 | patches core logic instead of an external edge |
| [S13](../catalog/semantic.md#s-series) | J6 | passes only via shared state a sibling set up |
| [S14](../catalog/semantic.md#s-series) | J2 | recorded model output as the oracle |
| [S15](../catalog/semantic.md#s-series) | J6 | hand-rolled retry/poll loop masking flakiness |
| [S16](../catalog/semantic.md#s-series) | J4 | call-verification as the sole oracle |
| [S17](../catalog/semantic.md#s-series) | J4 | exception-path oracle blindness |
| [S18](../catalog/semantic.md#s-series) | J3 | contract-impossible stub value |
| [S21](../catalog/semantic.md#s-series) | J2 | self-judging LLM/agent assertion |

## Patterns characteristic of each level { #by-level }

The same class of false-green appears at the level the test runs. These are the typical
clusters.

### Unit (the bulk of false-greens)

- always-true / tautology: `C5`, `C7`, `C52`, `JS30`
- no oracle / empty body: `C2`, `C2b`, `JS2`
- asserts its own double / mock: `C13b`, `C55`, `C11a`, `JS8`, `JS27`, `S8` (stub-echo), `S16`
- conditional-only / never runs: `C21`, `JS9`, `C20` (after a terminator), `JS25` (iterator over an empty collection)
- never-awaited coroutine: `C56`; bare comparison: `C59`; unconfigured Mock attr: `C57`
- semantic: `S5` (tests the framework), `S3` (plausible-but-wrong expected value), `S6`

### Integration (crosses the boundary: I/O, DB, HTTP, collaborator)

- request oracle turned off: `C9b` (`expected_status=any`)
- capture without asserting: `C50` (caplog / assertLogs)
- mocking the unit under test instead of the edge: `S12`
- a round-trip that only confirms what you sent (empty DB / HTTP liveness)
- semantic: `S9`, `S10`, `S11`, `S18` (stub with a value impossible under the contract)

### E2E (the full stack: browser, flow, hardware)

- sleep as synchronization instead of `Wait Until`: `C16`
- acts and only logs/screenshots without verifying: `C2b` in browser/login, `R4` (`No Operation` only)
- forced-green: `R1` (`Pass Execution`), `R2` (hollow verifier keyword)
- element presence or `Should Be True` on a string literal as the sole oracle: `R6`
- semantic: `S1` (intent mismatch: the name promises what the body does not verify), `S2` (irrelevant oracle)

### Diagnostics (off by default, not false-green)

`D1`, `D3`-`D8`, `M2` are hygiene and maintainability (control flow, long test). They turn on
with `--diagnostics`. They are not the false-green thesis; see
[what we do not flag](what-we-do-not-flag.md).
