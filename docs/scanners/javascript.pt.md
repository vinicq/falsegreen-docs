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

## First finding

Salve um teste cuja asserção nunca roda:

```ts
// demo.test.ts
import { add } from "./add";

it("adds", () => {
  expect(add(2, 2));   // sem matcher - isso não asserta nada
});
```

Rode o scanner sobre ele:

```bash
npx falsegreen-js demo.test.ts
```

Ele reporta:

```
demo.test.ts:5  [JS2] expect() with no matcher - the assertion never runs
    level: unit   fix: add a matcher, e.g. .toBe(4)

Summary: 1 high, 0 low.
```

## Reading a finding

Cada linha carrega os mesmos campos:

- `demo.test.ts:5` - o arquivo e a linha que disparou o achado.
- `[JS2]` - o código do catálogo. `JS2` é o `expect()` sem matcher. Cada código
  está explicado no [catálogo JS/TS](../catalog/javascript-typescript.md).
- `level: unit` - em qual nível da [pirâmide de testes](../concepts/pyramid.md) o
  arquivo está, lido a partir dos imports do arquivo.
- `fix:` - uma dica de uma linha. Aqui: adicione um matcher para a asserção
  realmente rodar.

`--format json|sarif|junit` dá um relatório legível por máquina; o SARIF sobe
para o GitHub code scanning e os achados aparecem direto no pull request.

## Complete usage and configuration { #complete-usage-and-configuration }

O guia inicial acima é o caminho de cinco minutos. Esta seção é a referência completa: cada
canal de instalação, cada formato de saída, cada chave de configuração, o contrato de código de
saída e a integração com CI. Espelha o que o [README do projeto](https://github.com/vinicq/falsegreen-js)
documenta.

### Install channels { #install-channels }

Node 18 ou mais novo. O scanner lê `.js`, `.jsx`, `.ts`, `.tsx`, `.mjs`, `.cjs`, `.mts`, `.cts`.

```bash
npm install -D falsegreen-js   # dependência de desenvolvimento do projeto
npm install -g falsegreen-js   # instalação global
npx falsegreen-js .            # roda a última versão do npm sem instalar
node dist/cli.js .             # roda de um checkout clonado
```

Agnóstico de runner: o vocabulário de asserção e teste cobre Jest, Vitest, Mocha + Chai, Jasmine,
AVA, `node:test`, tap, Cypress, Playwright, Testing Library e Vue Test Utils, então um teste Mocha
ou AVA não é confundido com um que nunca checa nada.

### Invocation { #invocation }

```bash
npx falsegreen-js                 # escaneia o cwd
npx falsegreen-js src test        # escaneia caminhos
npx falsegreen-js --staged        # só os arquivos de teste no stage do git (pre-commit)
```

Cada achado é reportado com seu [nível na pirâmide](../concepts/pyramid.md) (unit / integration /
e2e, lido dos imports do arquivo) e uma dica de correção de uma linha; o resumo separa os achados
por nível e lista as correções mais comuns.

### Output formats { #output-formats }

`--format text|json|sarif|junit` seleciona o formato do relatório (padrão `text`). `--json`
continua como alias de `--format json`. Eles casam com o [irmão Python](python.md) conceito a
conceito, então um pipeline pode trocar um scanner pelo outro.

```bash
npx falsegreen-js . --json                # JSON legível por máquina
npx falsegreen-js . --format sarif        # SARIF 2.1.0
npx falsegreen-js . --format junit        # JUnit XML
npx falsegreen-js . --output report.json  # grava em arquivo
npx falsegreen-js . --output .falsegreen/ # grava report.<ext> num diretório
```

- **`sarif`** emite SARIF 2.1.0: uma regra por código presente, um resultado por achado, `error`
  para ALTO, `warning` para BAIXO, `note` para desligado. As tags do resultado carregam o
  julgamento (J1-J6), o grupo de risco (`risk:effectiveness`...) e o nível. Suba para o GitHub code
  scanning para ver os achados inline no PR.
- **`junit`** emite JUnit XML: achados ALTO viram `<failure>`, o resto vira `<skipped>`, então um
  reporter de CI os mostra como suíte falhando.

O relatório JSON ainda carrega `riskGroup` e `oracleRegistryVersion`: os códigos são classificados
em grupos de risco fechados, e o registro de oráculos (sync-fail / promise / runner-registered /
value-only) é versionado para que a detecção async seja principiada, não casada por string.
`--output` aceita arquivo ou diretório; mantenha o diretório de saída no gitignore.

### Configuration { #configuration }

**Desabilitar / habilitar códigos (CLI).**

```bash
npx falsegreen-js --disable C7,JS3   # desliga códigos específicos
npx falsegreen-js --enable D8,M2     # reativa códigos desligados/opcionais na severidade do catálogo
npx falsegreen-js --diagnostics      # inclui o grupo de manutenibilidade D*/M* como avisos
```

`--enable` liga um código desligado por padrão na severidade do catálogo; não pode subir um código
acima do catálogo. Um código passado a `--enable` e `--disable` ao mesmo tempo continua desligado,
`--disable` vence.

**Grupo de diagnóstico (desligado por padrão).** Estes não são false-green, então só avisam: `D1`
assertion roulette, `D3` assert duplicado, `D4` casos de `each` só por índice, `D6` `console.*` num
teste, `D7` teste anônimo, `D8` número mágico, `M2` corpo de teste longo. Ligue todos com
`--diagnostics`, ou por código via `severity` na config.

**Supressão inline.** Um comentário na linha do achado silencia um achado justificado:

```ts
expect(user.id).toBe(user.id); // falsegreen: ignore[C7]
expect(x);                     // falsegreen: ignore
```

**Arquivo de config do projeto.** `falsegreen.json`, `.falsegreenrc.json`, ou uma chave
`"falsegreen"` no `package.json`:

```json
{
  "disable": ["C8"],
  "exclude": ["**/legacy/**"],
  "severity": { "JS3": "off", "C16": "high" }
}
```

Precedência, da maior para a menor: `--disable` na CLI, `--enable` na CLI, `disable` / `severity`
da config, padrão do catálogo.

**Auditoria de config.** `--config-audit` é um modo separado. Em vez de escanear arquivos de teste,
lê a config de Jest / Vitest (campo `jest` do `package.json`, `jest.config.*`, `vitest.config.*`) e
reporta as formas de nível de projeto pelas quais uma suíte fica verde por configuração:

- `PL10` - `passWithNoTests` passa numa execução vazia ou filtrada até o nada.
- `PL7` - sem `coverageThreshold` / `coverage.thresholds`.
- `PL8` - `bail` interrompe a execução cedo.

**Baseline (catraca).** Adote o scanner numa base grande sem corrigir todo achado legado de uma vez:

```bash
npx falsegreen-js --write-baseline   # registra os achados atuais em .falsegreen-baseline.json, exit 0
npx falsegreen-js --baseline         # reporta e falha só nos achados que não estão no baseline
```

`--baseline [PATH]` e `--write-baseline [PATH]` usam `.falsegreen-baseline.json` por padrão. A
identidade de um achado é um fingerprint de conteúdo (sha1 de caminho relativo + código + detalhe,
sem número de linha), então sobrevive a deslocamentos de linha não relacionados. Comite o baseline
e deixe o CI bloquear só nos achados novos.

### Exit codes { #exit-codes }

| Código | Significado |
|---|---|
| `0` | limpo, nenhum achado que afete o gate |
| `10` | só achados de baixa confiança |
| `20` | ao menos um achado de alta confiança |

Bloqueie o commit no `20`.

### CI integration { #ci-integration }

**GitHub Actions.** Um job que falha no exit `20`:

```yaml
name: falsegreen-js
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: "20" }
      - run: npx falsegreen-js .   # exit 20 falha o job
```

**Upload de SARIF para o GitHub code scanning.** Emita SARIF e passe para a action do CodeQL para
que os achados apareçam inline no pull request:

```yaml
      - run: npx falsegreen-js . --format sarif --output falsegreen.sarif
        continue-on-error: true
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: falsegreen.sarif
```

**Hook de pre-commit.** Rode o scan dos arquivos em stage como hook com `--staged`:

```yaml
  - repo: local
    hooks:
      - id: falsegreen-js
        name: falsegreen-js
        entry: npx falsegreen-js --staged
        language: system
        pass_filenames: false
        types_or: [javascript, ts, tsx, jsx]
```

Para a camada que nenhum scan estático alcança (um teste verde falha quando o código está errado?),
rode um teste de mutação como o [Stryker](https://stryker-mutator.io/). O falsegreen-js é o
pré-filtro barato em todo commit; os casos semânticos (intenção, correção do oráculo) são a
[falsegreen-skill](skill.md).

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
