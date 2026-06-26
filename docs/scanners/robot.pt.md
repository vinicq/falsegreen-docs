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
