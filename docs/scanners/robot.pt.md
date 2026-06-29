# robotframework-falsegreen (Robot Framework)

[![CI](https://github.com/vinicq/robotframework-falsegreen/actions/workflows/ci.yml/badge.svg)](https://github.com/vinicq/robotframework-falsegreen/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/robotframework-falsegreen.svg)](https://pypi.org/project/robotframework-falsegreen/)
[![Downloads](https://img.shields.io/pypi/dm/robotframework-falsegreen.svg)](https://pypistats.org/packages/robotframework-falsegreen)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/vinicq/robotframework-falsegreen/blob/main/LICENSE)

O scanner determinístico para Robot Framework. Uma varredura estática sobre o parser oficial
(`robot.api.get_model`), sem execução. Ele reconhece o vocabulário de verificação por todo o
ecossistema de bibliotecas do Robot, de modo que uma checagem real não seja confundida com "sem
oráculo".

- Repositório: [github.com/vinicq/robotframework-falsegreen](https://github.com/vinicq/robotframework-falsegreen)
- Catálogo: [códigos Robot Framework](../catalog/robot.md)
- Comando da CLI: `rffalsegreen`

## Install

```bash
pip install robotframework-falsegreen
```

## Use

```bash
rffalsegreen path/to/suite              # scan
rffalsegreen --format json|sarif|junit  # output shape
rffalsegreen --config-audit             # robot.toml / invocation for project-level checks
```

## First finding

Salve uma suíte cujo único passo é um no-op:

```robotframework
*** Test Cases ***
Login Works
    No Operation
```

Rode o scanner sobre ela:

```bash
rffalsegreen demo.robot
```

Ele reporta:

```
demo.robot:3  [R4] No Operation as the only step - the test verifies nothing
    level: e2e   fix: call a real verification keyword (Should Be Equal, status check)

Summary: 1 high, 0 low.
```

## Reading a finding

Cada linha carrega os mesmos campos:

- `demo.robot:3` - o arquivo e a linha que disparou o achado.
- `[R4]` - o código do catálogo. `R4` é um teste só com `No Operation`. Cada
  código está explicado no [catálogo Robot](../catalog/robot.md).
- `level: e2e` - em qual nível da [pirâmide de testes](../concepts/pyramid.md) a
  suíte está, lido a partir da library importada em `*** Settings ***`.
- `fix:` - uma dica de uma linha. Aqui: chame um keyword de verificação real.

`--format json|sarif|junit` dá um relatório legível por máquina para CI.

## Complete usage and configuration { #complete-usage-and-configuration }

O guia inicial acima é o caminho de cinco minutos. Esta seção é a referência completa: cada
canal de instalação, cada formato de saída, cada chave de configuração, o contrato de código de
saída e a integração com CI. Espelha o que o [README do projeto](https://github.com/vinicq/robotframework-falsegreen)
documenta.

### Install channels { #install-channels }

```bash
pip install robotframework-falsegreen   # comando da CLI: rffalsegreen
```

O scanner faz uma passagem estática sobre o parser oficial do Robot (`robot.api.get_model`); nunca
executa uma suíte, e se restringe a arquivos `.robot` / `.resource`.

### Invocation { #invocation }

```bash
rffalsegreen                  # escaneia o cwd
rffalsegreen tests/           # escaneia um caminho
rffalsegreen --disable C16    # desliga códigos específicos
```

Cada achado é reportado com seu [nível na pirâmide](../concepts/pyramid.md) (unit / integration /
e2e, lido das bibliotecas importadas pela suíte) e uma dica de correção de uma linha; o resumo em
texto separa os achados por nível e lista as correções mais comuns.

### Output formats { #output-formats }

`--format text|json|sarif|junit` seleciona o formato do relatório (padrão `text`). `--json`
continua como alias de `--format json`.

```bash
rffalsegreen --format json            # JSON legível por máquina
rffalsegreen --format sarif           # SARIF 2.1.0
rffalsegreen --format junit           # JUnit XML
rffalsegreen --output report.sarif    # grava em arquivo
rffalsegreen --output .falsegreen/    # grava report.<ext> num diretório
```

- **`sarif`** emite SARIF 2.1.0 (nome da ferramenta `robotframework-falsegreen`): ALTO mapeia para
  `error`, BAIXO para `warning`, desligado/info para `note`, e cada resultado é marcado com sua
  família de julgamento e nível da pirâmide para que o GitHub code scanning agrupe e filtre os
  achados.
- **`junit`** emite JUnit XML: um testcase por achado, ALTO vira `<failure>`, o resto vira
  `<skipped>`.

`--json` mantém seu envelope (`tool` / `version` / `judgments` / `findings`). `--output` aceita
arquivo ou diretório; um caminho sem extensão ou com barra final recebe `report.<ext>` (para JUnit,
`report.xml`). Mantenha o diretório de saída no gitignore.

O reconhecedor de verificação conhece a convenção `Should` mais as formas de biblioteca
(SeleniumLibrary, o motor de asserção do Browser, RequestsLibrary incluindo
`expected_status=<code>`, RESTinstance, DatabaseLibrary), então uma asserção de API ou UI não é
lida como sem-oráculo.

### Configuration { #configuration }

**Desabilitar códigos (CLI).** `--disable C16` desliga códigos específicos numa execução.

**Supressão inline.** Um comentário na linha do achado silencia um achado justificado sem desligar
o código na suíte inteira. O token e a sintaxe de colchetes casam com falsegreen (Python) e
falsegreen-js:

```robotframework
*** Test Cases ***
Polls A Real Service
    Sleep    1s    # falsegreen: ignore[C16]      # silencia só o C16 nesta linha
    Should Be Equal    ${result}    ${expected}
```

`# falsegreen: ignore` (sem colchetes) silencia todos os códigos na linha; `ignore[C16,C20]`
silencia só os códigos listados. Só o token exato `falsegreen:` suprime; um `# ignore` puro não.

**Grupo de diagnóstico (desligado por padrão).** Os códigos de manutenibilidade (`D2`, `M2`) não
são false-green, então são opcionais; ligue-os por código via `severity` na config.

**Auditoria de config.** `--config-audit` é um modo separado. Em vez de escanear suítes, lê a
config de execução do Robot (`robot.toml`, `pyproject.toml` `[tool.robot]` e arquivos de argumento
`*.args` encontrados recursivamente, pulando diretórios ignorados como `results/` / `output/`) e
reporta `PL9`: uma opção `--skiponfailure` / `--noncritical` que transforma um teste falho num
passe não-fatal (legado, removido no RF 4+). O scan por arquivo não vê a config de execução.

**Baseline (adotar numa suíte que já tem achados).** Registre um baseline e depois falhe só nos
achados novos:

```bash
rffalsegreen --write-baseline    # registra os achados atuais em .falsegreen-baseline.json
rffalsegreen --baseline          # escaneia, suprimindo tudo que está no baseline
```

Ambas as flags aceitam um caminho opcional (padrão `.falsegreen-baseline.json`). O baseline
fingerprinta um achado por conteúdo (sha1 de caminho relativo, código, nome do teste/keyword,
detalhe, sem número de linha), então sobrevive a edições que deslocam um teste no arquivo. Comite o
baseline para que o CI veja o mesmo conjunto.

### Exit codes { #exit-codes }

| Código | Significado |
|---|---|
| `0` | limpo, nenhum achado que afete o gate |
| `10` | só achados de baixa confiança |
| `20` | ao menos um achado de alta confiança |

Ligue o exit `20` ao CI para bloquear o merge.

### CI integration { #ci-integration }

**GitHub Actions.** Um job que falha no exit `20`:

```yaml
name: robotframework-falsegreen
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.x" }
      - run: pip install robotframework-falsegreen
      - run: rffalsegreen tests/   # exit 20 falha o job
```

**Upload de SARIF para o GitHub code scanning.** Emita SARIF e passe para a action do CodeQL para
que os achados apareçam inline no pull request:

```yaml
      - run: rffalsegreen tests/ --format sarif --output falsegreen.sarif
        continue-on-error: true
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: falsegreen.sarif
```

**Hook de pre-commit.** Adicione ao `.pre-commit-config.yaml`:

```yaml
  - repo: https://github.com/vinicq/robotframework-falsegreen
    rev: v0.3.0
    hooks:
      - id: rffalsegreen
```

O hook se restringe a arquivos `.robot` / `.resource` e passa os caminhos em stage ao scanner,
então nunca reescaneia `results/` / `output/`. Ele respeita os códigos de saída, então um achado
ALTO falha o commit. Rode sob demanda contra os arquivos em stage com `pre-commit run rffalsegreen`.

O Robot não tem um teste de mutação padrão, então a camada de runtime (um teste verde falha quando
o código está errado?) é revisão manual; os casos de intenção são a [falsegreen-skill](skill.md).

## What it covers

Detalhe completo por código no [catálogo Robot](../catalog/robot.md).

| Grupo | Códigos | Efeito |
|---|---|---|
| **Compartilhado com Python** (F1-F6) | `C2`, `C2b`, `C3`, `C5`, `C6`, `C7`, `C9`, `C16`, `C20`, `C21`, `C23`, `C32`, `C37`, `CC` | ALTO bloqueia, BAIXO avisa |
| **Específico de Robot** | `R1` (Pass Execution), `R2` (verificador oco), `R3` (test cases em .resource), `R4` (No Operation), `R5` (template vazio), `R6` (Should Be True sobre uma string), `R7` (keyword de template oca) | idem |
| **Diagnóstico** (F8) | `D2`, `M2` | opcional |
| **Projeto / CI** (`--config-audit`) | `PL9` (`--skiponfailure`/noncritical legados via robot.toml/args) | lê a config |

O reconhecedor de verificação conhece a convenção `Should` mais as formas das bibliotecas
(SeleniumLibrary, motor de asserção do Browser, RequestsLibrary incluindo
`expected_status=<code>`, RESTinstance, DatabaseLibrary), então uma asserção de API ou de UI não é
lida errado como sem-oráculo.

## What it does not cover, and why

### Fora de escopo (o eixo errado)
Mesmo limite da família. Veja [cobertura vs a literatura](../concepts/denominator.md).

### Deixado de fora de propósito
| O quê | Por que não |
|---|---|
| **`Wait Until Keyword Succeeds`** como máscara de flakiness (catálogo RF16) | retry legítimo em torno de flakiness genuína de E2E; falso positivo alto |
| **Estado compartilhado entre testes** (`Set Suite/Global Variable` lido por um irmão) | exige análise de ordenação da suíte inteira, não um fato por arquivo; o próprio guia HowTo até autoriza alguma dependência entre testes. Falso positivo alto |
| **`C31` - valor capturado nunca usado** (`${x}= Get Text` nunca lido) | uma lacuna real de recall, mas a superfície de falso-positivo (valor usado só em `Log`, numa string de `Evaluate`, ou em teardown) precisa de limites cuidadosos. Adiado para uma segunda passagem ([issue #34](https://github.com/vinicq/robotframework-falsegreen/issues/34)) |
| **Keyword de usuário morta** (catálogo RF6) | exige uma passagem de projeto entre arquivos; o scanner é de arquivo único hoje |
| **Higiene pura** (nomenclatura ruim, taxa de comentários, listas longas de parâmetros) | o Robocop é dono do guia de estilo do Robot; o scanner não o duplica |

### Sem equivalente limpo no Robot
Alguns códigos irmãos de Python/JS não têm forma fiel no Robot e são pulados de propósito em vez
de forçados. O catálogo os anota explicitamente.

### Além do scanner
Smells de runtime (Test Run War, dependência de ordem entre suítes) precisam da suíte rodando; a
fatia semântica é a [falsegreen-skill](skill.md). Veja [escopo e honestidade](../concepts/scope.md).
