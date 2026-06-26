# falsegreen-js (JavaScript / TypeScript)

[![CI](https://github.com/vinicq/falsegreen-js/actions/workflows/ci.yml/badge.svg)](https://github.com/vinicq/falsegreen-js/actions/workflows/ci.yml)
[![npm](https://img.shields.io/npm/v/falsegreen-js.svg)](https://www.npmjs.com/package/falsegreen-js)
[![Downloads](https://img.shields.io/npm/dm/falsegreen-js.svg)](https://www.npmjs.com/package/falsegreen-js)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/vinicq/falsegreen-js/blob/main/LICENSE)

O scanner determinístico para JS/TS. Uma varredura estática via TypeScript compiler API, sem
execução de código, agnóstica ao runner entre Jest, Vitest, Mocha+Chai, Jasmine, AVA, node:test,
Cypress, Playwright e Testing Library. Cobre `.js`, `.jsx`, `.ts`, `.tsx`, `.mjs`, `.cjs`, `.mts`,
`.cts`.

- Repositório: [github.com/vinicq/falsegreen-js](https://github.com/vinicq/falsegreen-js)
- Catálogo: [códigos JavaScript / TypeScript](../catalog/javascript-typescript.md)

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

O relatório JSON carrega `riskGroup` e `oracleRegistryVersion`: os códigos são classificados em
grupos de risco fechados, e o registro de oráculos (sync-fail / promise / runner-registered /
value-only) é versionado, de modo que a detecção de assíncrono é baseada em princípios em vez de
casada por string.

## What it covers

Detalhe completo por código no [catálogo JS/TS](../catalog/javascript-typescript.md).

| Grupo | Códigos | Efeito |
|---|---|---|
| **Compartilhado com Python** (F1-F6) | `C2`, `C2b`, `C5`, `C6`, `C7`, `C8`, `C9`, `C16`, `C18`, `C20`, `C21`, `C23`, `C37`, `C44`, `CC` | ALTO bloqueia, BAIXO avisa |
| **Específico de JavaScript** | `JS1`-`JS9`, `JS11`, `JS13`, `JS15`, `JS17`, `JS18`, `JS21`, `JS22` | idem |
| **Diagnóstico / acoplamento** (F8) | `D1`, `D3`, `D4`, `D6`, `D7`, `D8`, `M2` | opcional |
| **Projeto / CI** (`--config-audit`) | `PL10` (`--passWithNoTests`), `PL7` (limiar de cobertura), `PL8` (bail) | lê a config do jest/vitest |

## What it does not cover, and why

### Fora de escopo (o eixo errado)
O mesmo limite do resto da família: fragilidade/falso-vermelho, higiene, lentidão, design,
nomenclatura, duplicação, runtime. Veja [cobertura vs a literatura](../concepts/denominator.md).

### Códigos JS deixados de fora de propósito
| Código | O que sinalizaria | Por que não |
|---|---|---|
| **JS10** | qualquer condicional no corpo do teste | amplo demais; `jest/no-conditional-in-test` (ESLint) já cobre. No máximo, diagnóstico |
| **JS12** | uma promise com `expect` não retornada/aguardada | absorvido por `JS7` |
| **JS14** | um snapshot gigante | higiene (F8), não false-green |
| **JS16** | teste assíncrono sem a guarda `expect.assertions(n)` | a ausência de uma guarda não é um smell; sinalizá-la tem taxa de falso-positivo altíssima |
| **JS19** | `toBe` sobre um literal de objeto/array (era `toEqual`) | isto é falso-VERMELHO (uma checagem de identidade que falha em código correto): o eixo oposto |
| **JS20** | uma Promise comparada sem `resolves`/`rejects` | saber que o sujeito é uma Promise exige informação de tipo; falso positivo alto |

### Não aplicável a partir do Python
Códigos específicos do pytest não têm análogo em JS: as regras de coleta (família `C4`), os
códigos de binding/escopo de `pytest.raises`, os códigos de `capsys`/`os.environ`. Estão ausentes
de propósito, não esquecidos.

### Além do scanner
A fatia semântica (intenção, correção do oráculo) é a [falsegreen-skill](skill.md); o runtime
(mutação com Stryker) fica fora de banda. Veja [escopo e honestidade](../concepts/scope.md).
