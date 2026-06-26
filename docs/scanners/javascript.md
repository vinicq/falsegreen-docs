# falsegreen-js (JavaScript / TypeScript)

The deterministic JS/TS scanner. A static scan via the TypeScript compiler API, no code
execution, runner-agnostic across Jest, Vitest, Mocha+Chai, Jasmine, AVA, node:test, Cypress,
Playwright, and Testing Library. Covers `.js`, `.jsx`, `.ts`, `.tsx`, `.mjs`, `.cjs`, `.mts`,
`.cts`.

- Repository: [github.com/vinicq/falsegreen-js](https://github.com/vinicq/falsegreen-js)
- Catalog: [JavaScript / TypeScript codes](../catalog/javascript-typescript.md)

## Install

```bash
npm install --save-dev falsegreen-js
```

## Use

```bash
npx falsegreen-js src                       # scan
npx falsegreen-js --format json|sarif|junit # output shape (matches the Python sibling)
npx falsegreen-js --baseline .falsegreen-baseline.json
npx falsegreen-js --config-audit            # jest/vitest config for project-level false-green
```

The JSON report carries `riskGroup` and `oracleRegistryVersion`: codes are classified into closed
risk groups, and the oracle registry (sync-fail / promise / runner-registered / value-only) is
versioned so async detection is principled rather than string-matched.

## Scope

Structural slice only; the semantic slice is [falsegreen-skill](skill.md). Codes share an id with
[falsegreen](python.md) where the concept matches. See
[scope and honesty](../concepts/scope.md).
