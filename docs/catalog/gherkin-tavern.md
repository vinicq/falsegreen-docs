# Gherkin and Tavern

Two DSL formats the [skill](../scanners/skill.md) covers as a semantic, text-based pass. The
`.feature` and `*.tavern.yaml` files are not code, so the static scanners do not parse them; the
skill reads the scenario or stage and maps each finding to [J1-J6](../concepts/judgments.md).

## Gherkin / BDD (`.feature`)

Used by Cucumber.js, behave, pytest-bdd, SpecFlow. The step definitions live in `.js`/`.ts`/`.py`
and are covered by the static scanners; this pass is over the scenarios themselves. The `Then`
step is the oracle.

| Pattern | J | What it detects | Mirrors |
|---|---|---|---|
| Scenario with no `Then` | J1 | only `Given`/`When` steps; never states an expected outcome | C2b |
| `Then` that does not verify | J4 | the step definition only acts or logs, never asserts | C2b |
| Empty `Scenario Outline` | J1 | an `Examples:` table with no rows runs zero times | C2 |
| Tautological `Then` | J2 | asserts a constant (`Then 1 equals 1`) | C5 |

=== "BAD"
    ```gherkin
    Scenario: User logs in
      Given a registered user
      When they submit valid credentials
      # no Then - nothing is verified
    ```
=== "CLEAN"
    ```gherkin
    Scenario: User logs in
      Given a registered user
      When they submit valid credentials
      Then they see the dashboard
    ```

**Look-alikes - do NOT flag:** a `Then` whose step definition does assert (UI presence counts at
the E2E layer); `@skip`/`@wip`/`@manual` tagged scenarios (intentionally not run, report as
skipped, low).

## Tavern (`*.tavern.yaml`)

A pytest plugin for HTTP/MQTT API testing. The test is YAML; the `response:` block is the oracle.

| Pattern | J | What it detects | Mirrors |
|---|---|---|---|
| `request:` with no `response:` | J1/J4 | the call is sent but nothing is verified | C2b |
| `response:` checks only `status_code` | J4 | "something came back" when the body is what matters | C6 |
| Overly broad status acceptance | J4 | a `status_code` list/range that accepts almost any outcome | C6 |
| Non-asserting `verify_response_with` | J3/J4 | an external function that does not verify | C2b |

=== "BAD"
    ```yaml
    stages:
      - name: create user
        request:
          url: http://localhost/users
          method: POST
        # no response - the stage passes as long as the request does not error
    ```
=== "CLEAN"
    ```yaml
    stages:
      - name: create user
        request:
          url: http://localhost/users
          method: POST
        response:
          status_code: 201
          json:
            name: Alice
    ```

**Look-alikes - do NOT flag:** a setup-only stage followed by a later stage that validates the
response; a `response:` with a `json:` body match or `$ext` schema validation (a real oracle).

## Visual testing

A note that applies across runners: a test whose only check is `percySnapshot()` /
`cy.percySnapshot()` verifies nothing locally - the diff is approved by a human in a dashboard,
asynchronously - so it reads as no-verification (C2b-equivalent). A Playwright/Storybook
`toHaveScreenshot()` baseline is generated from output, so if it is the only assertion it is
snapshot-only (JS3 / C14). A visual snapshot *alongside* a behavioral assertion is fine.
