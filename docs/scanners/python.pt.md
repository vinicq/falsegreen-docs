# falsegreen (Python)

[![CI](https://github.com/vinicq/falsegreen/actions/workflows/ci.yml/badge.svg)](https://github.com/vinicq/falsegreen/actions/workflows/ci.yml)
[![PyPI](https://img.shields.io/pypi/v/falsegreen.svg)](https://pypi.org/project/falsegreen/)
[![Downloads](https://img.shields.io/pypi/dm/falsegreen.svg)](https://pypistats.org/packages/falsegreen)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/vinicq/falsegreen/blob/main/LICENSE)

O scanner determinístico para Python/pytest. Uma passagem AST sem dependências que valida cada
teste contra os códigos de falso-positivo que um parser consegue provar. Achados ALTO bloqueiam o
commit, os BAIXO avisam, e um grupo de diagnóstico/acoplamento é opcional.

- Repositório: [github.com/vinicq/falsegreen](https://github.com/vinicq/falsegreen)
- Catálogo: [códigos Python](../catalog/python.md)

## Install

```bash
pip install falsegreen
```

## Use

```bash
falsegreen path/to/tests        # scan
falsegreen --staged             # only staged files (pre-commit)
falsegreen --json               # machine-readable report
falsegreen --diagnostics        # include the opt-in F8 group
falsegreen --config-audit       # read pytest/coverage config for project-level false-green
```

Achados ALTO encerram com código diferente de zero, então a ferramenta entra em CI e pre-commit
sem ajuste. O relatório numera cada achado com seu código, julgamento, nível da pirâmide,
localização, evidência e uma dica de correção.

## What it covers

O scanner mais completo da família: é a referência que os outros espelham. O detalhe completo por
código está no [catálogo Python](../catalog/python.md).

| Grupo | Códigos | Efeito |
|---|---|---|
| **Falso-positivo** (F1-F6) | ~45 códigos `C*` ativos + `CC` | ALTO bloqueia, BAIXO avisa |
| **Diagnóstico / acoplamento** (F8) | `D1`, `D3`, `D4`, `D5`, `D6`, `M2` | opcional, nunca bloqueia |
| **Projeto / CI** (F5, `--config-audit`) | `PL2` (filterwarnings não está em error), `PL7` (sem `--cov-fail-under`), `PL8` (`-x`/`--maxfail` mascara a contagem) | lê a config, reporta |

## What it does not cover, and why

### Fora de escopo (o eixo errado)
Fragilidade/falso-vermelho, higiene, lentidão, design, nomenclatura, duplicação, runtime/cultura
não são false-green. Veja [cobertura vs a literatura](../concepts/denominator.md) para o limite
completo.

### Códigos deixados de fora de propósito
Estes foram avaliados contra o catálogo consolidado e deixados de fora, cada um por um motivo.
Deixá-los de fora é a política de precisão em primeiro lugar: um falso positivo é pior que um
escape.

| Código | O que sinalizaria | Por que não |
|---|---|---|
| **C40** | `assert mock.attr` sem spec (sempre verdadeiro) | sem análise de spec a taxa de falso-positivo é alta; o conceito vive na skill (F7) |
| **C46** | rede/DB real sem dublê (`requests`, `socket`) | legítimo num teste de integração; sinalizá-lo exige saber o nível, então roteia para a skill / `--config-audit` |
| **C47** | asserção depende da ordenação de dict/set | falso positivo alto (a maioria das coleções é determinística no uso); em vez disso, uma nota na skill |

### Reservados para a passagem semântica (F7)
Mockar a unidade sob teste (caso 10), asserir o valor passado ao mock (caso 11), reimplementar a
fórmula de produção (caso 12), um valor esperado que contradiz a intenção (caso 18), estado
compartilhado emprestado (caso 15). Nenhum AST prova intenção ou fluxo interprocedural. Estes
vivem na [falsegreen-skill](skill.md). `C14` (snapshot da própria saída do código) é o único canto
codificável.

### Precisa de runtime (não prometido estaticamente)
`python -O` removendo `assert`, um erro de coleta reportado como "0 tests passed", um passo de CI
que roda um subconjunto e reporta verde (`PL1`/`PL4`/`PL6`). O `PL1` agora tem uma fatia
detectável por configuração: o `--config-audit` sinaliza `python -O`/`-OO` ou `PYTHONOPTIMIZE=1`
em `tox.ini`/`addopts` do pytest como aviso de nível de projeto. O resto só aparece quando a suíte
roda; são documentados, não reivindicados. O caminho honesto é teste de mutação (mutmut,
cosmic-ray), que fica fora de banda.

Veja [escopo e honestidade](../concepts/scope.md) para o limite entre as camadas.
