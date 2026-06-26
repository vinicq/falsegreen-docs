# Test pyramid and levels

The scanners discover a test file in a level-agnostic way, but a few codes read differently
depending on the level. Knowing the level avoids a false positive: what is a weak assertion in
a unit test is the correct assertion in an end-to-end test.

| Level | Oracle | What real coverage looks like |
|---|---|---|
| **Unit** | doubles at the boundaries; `assert` / `expect` on the return value | the unit's own logic, in isolation |
| **Integration (API + DB)** | the real response or row is the verification | a live client or store, no double |
| **E2E** | a page element or state is present | a browser or device driver |

## The level is read from signals, not guessed

The tools read the level from concrete cues, with a fixed precedence (strongest signal wins):

1. A **doubled boundary** keeps the test at unit/component even if a real client is imported.
   The mock *is* the boundary.
2. Otherwise a **real boundary** (a live HTTP client, an ORM, a database driver) makes it
   integration.
3. Otherwise a **browser or device driver** makes it E2E.
4. No signal at all means unit.

A `conventions:` block overrides all of this. Markers like `smoke`, `slow`, `asyncio`, or
`anyio` are level-neutral.

## The level-aware exemptions

Some codes hold back once the level is clear, so they do not fire on the correct pattern:

- **`C6`** (weak truthiness check) does not fire on `assert response` in an HTTP test or
  `assert locator` in a Playwright test. At those layers, presence *is* the contract.
- **`C14`** (golden file from output) does not fire in browser snapshot testing, where a
  generated baseline is the intended workflow.
- **`C16`** (uncontrolled time) does not fire when `freezegun` or `time-machine` is imported,
  because the clock is doubled.

## The mother rule

A real database or API call inside a test that calls itself a unit test is, by itself, the
smell. It is not a question of category: it is mystery guest, resource optimism, the inverse of
over-mocking. A database is integration when it hits a real datastore, even an in-memory SQLite
or a testcontainer; it is unit when the data layer is doubled.

The static proxies for "real I/O in a unit test" are `C23` (hard-coded path or IP URL, in all
three scanners) and `C29` / `C30` in Python. The [skill](../scanners/skill.md) reads the level
directly and relaxes or tightens the oracle accordingly.
