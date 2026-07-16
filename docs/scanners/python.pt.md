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

## First finding

Salve um teste que sempre passa:

```python
# test_demo.py
def test_demo():
    assert True
```

Rode o scanner sobre ele:

```bash
falsegreen test_demo.py
```

Ele reporta:

```
test_demo.py:3  [C5] always-true check (assert True / tuple / or True)
    level: unit   fix: assert the real behaviour, not a constant or tautology

Summary: 1 high, 0 low.
```

## Reading a finding

Cada linha carrega os mesmos campos:

- `test_demo.py:3` - o arquivo e a linha que disparou o achado.
- `[C5]` - o código do catálogo. `C5` é a checagem sempre-verdadeira. Cada código
  está explicado no [catálogo Python](../catalog/python.md).
- `level: unit` - em qual nível da [pirâmide de testes](../concepts/pyramid.md) o
  arquivo está; isso muda o que conta como checagem de verdade.
- `fix:` - uma dica de uma linha. Aqui: asserte o comportamento real, não uma
  constante.

Códigos de saída integram em CI: `0` limpo, `10` só baixa confiança, `20` ao
menos um achado de alta confiança. Bloqueie o build no `20`.

## Complete usage and configuration { #complete-usage-and-configuration }

O guia inicial acima é o caminho de cinco minutos. Esta seção é a referência completa: cada
canal de instalação, cada formato de saída, cada chave de configuração, o contrato de código de
saída e a integração com CI. Espelha o que o [README do projeto](https://github.com/vinicq/falsegreen)
documenta.

### Install channels { #install-channels }

Python 3.8 ou mais novo, sem dependências de runtime de terceiros.

```bash
pip install falsegreen          # instalação no projeto ou virtualenv
uvx falsegreen tests/           # roda uma vez sem instalar (uv)
pipx run falsegreen tests/      # roda uma vez sem instalar (pipx)
```

`python -m falsegreen ...` é equivalente ao comando `falsegreen` quando o ponto de entrada não
está no `PATH`.

### Invocation { #invocation }

```bash
falsegreen                        # escaneia o diretório atual
falsegreen tests/                 # escaneia uma pasta ou um arquivo
falsegreen --staged               # só os arquivos de teste no stage do git (pre-commit)
falsegreen --summary              # uma linha "N escaneados, M sinalizados" no stderr
```

O scanner lê apenas os arquivos de teste, nunca os importa nem os executa, então um teste
quebrado ou hostil não roda através dele. Cada achado carrega seu [nível na pirâmide](../concepts/pyramid.md)
(unit / integration / e2e, lido dos imports do arquivo) e uma dica de correção de uma linha; o
resumo em texto separa os achados por nível e lista as correções mais comuns.

### Output formats { #output-formats }

`--format text|json|sarif|junit` seleciona o formato do relatório (padrão `text`). `--json`
continua como alias de `--format json`.

```bash
falsegreen tests/ --json                  # JSON legível por máquina
falsegreen tests/ --format sarif          # SARIF 2.1.0
falsegreen tests/ --format junit          # JUnit XML
falsegreen tests/ --output report.sarif   # grava em arquivo
falsegreen tests/ --output .falsegreen/   # grava report.<ext> num diretório
```

- **`json`** carrega o envelope completo: tool, version, judgments e a lista de achados.
- **`sarif`** emite SARIF 2.1.0, mapeando ALTO para `error` e BAIXO para `warning`, para o GitHub
  code scanning e anotações inline no pull request.
- **`junit`** emite JUnit XML, onde achados ALTO viram `<failure>` para que um reporter de CI os
  mostre como suíte falhando.

`--output` aceita arquivo ou diretório: um caminho sem extensão ou com barra final (`.falsegreen/`)
recebe `report.<ext>` no formato escolhido. Relatórios são artefatos de execução, então mantenha o
diretório de saída no gitignore.

### Configuration { #configuration }

**Desabilitar códigos (CLI).** `--disable C6,C2b` desliga códigos específicos numa execução.

**Supressão inline.** Um comentário na linha do achado silencia um achado justificado sem
desligar o código na suíte inteira:

```python
assert user.id == user.id  # falsegreen: ignore[C7]   silencia só o C7 nesta linha
assert x                   # falsegreen: ignore        silencia todos os códigos nesta linha
```

Só o token exato `falsegreen:` suprime; um `# ignore` puro não.

**Arquivo de config do projeto.** `[tool.falsegreen]` no `pyproject.toml`, ou um `.falsegreen.toml`
plano na raiz do repo (`.falsegreen.toml` vence se ambos existirem). Aponte para um arquivo
específico com `--config PATH`.

```toml
[tool.falsegreen]
disable = ["C13b"]            # desliga estes códigos em todo lugar
exclude = ["tests/legacy/*"]  # ignora arquivos que casam com estes globs
long_test_threshold = 30      # limite de linhas para M2 (padrão: 50)
inline_setup_threshold = 3    # limite de statements para D5 (padrão: 5)

[tool.falsegreen.severity]
C8 = "high"    # promove: agora bloqueia o commit (exit 20)
C6 = "off"     # equivale a adicionar C6 ao disable
C22 = "low"    # habilita a checagem async-never-awaits
D1 = "info"    # habilita Assertion Roulette (diagnóstico, nunca bloqueia)
M2 = "info"    # habilita Long Test Method (diagnóstico)
```

Valores de `severity`: `high`, `low`, `info` ou `off`. Achados `info` aparecem nas seções
DIAGNOSTIC / COUPLING e não afetam o código de saída, o grupo opcional F8. As chaves
`long_test_threshold` e `inline_setup_threshold` ficam direto sob `[tool.falsegreen]`, não dentro
de `[severity]`. Precedência, da maior para a menor: `--disable` na CLI, inline
`# falsegreen: ignore`, o arquivo de config, o padrão embutido.

**Auditoria de config.** `--config-audit` é um modo separado. Em vez de escanear arquivos de
teste, lê a config de pytest e coverage do projeto (`pyproject.toml`, `pytest.ini`, `tox.ini`,
`setup.cfg`) e reporta as formas de nível de projeto pelas quais uma suíte fica verde por
configuração:

- `PL1` - `python -O` / `PYTHONOPTIMIZE` remove todo `assert` em runtime.
- `PL2` - `filterwarnings` não promove warnings a erros.
- `PL7` - sem trava de coverage (`--cov-fail-under` ausente).
- `PL8` - `addopts` interrompe a execução cedo com `-x` / `--maxfail`, mascarando a contagem.

**Baseline (adotar num repo legado).** Registre os achados que você já tem e falhe só nos novos:

```bash
falsegreen --write-baseline tests/   # grava .falsegreen-baseline.json, exit 0
falsegreen --baseline tests/         # suprime os achados registrados, falha nos novos
```

Um achado é fingerprintado por caminho relativo, código, detalhe e a linha-fonte normalizada
(não o número da linha), então adicionar código antes não redispara um achado já no baseline.
Comite `.falsegreen-baseline.json` e a catraca só aperta.

### Exit codes { #exit-codes }

| Código | Significado |
|---|---|
| `0` | limpo, nenhum achado que afete o gate |
| `10` | só achados de baixa confiança |
| `20` | ao menos um achado de alta confiança |

Bloqueie o build no `20`. O hook de pre-commit respeita o mesmo contrato.

### CI integration { #ci-integration }

**GitHub Actions.** Um job que falha no exit `20`:

```yaml
name: falsegreen
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: { python-version: "3.x" }
      - run: pip install falsegreen
      - run: falsegreen tests/   # exit 20 falha o job
```

**Upload de SARIF para o GitHub code scanning.** Emita SARIF e passe para a action do CodeQL para
que os achados apareçam inline no pull request:

```yaml
      - run: falsegreen tests/ --format sarif --output falsegreen.sarif
        continue-on-error: true
      - uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: falsegreen.sarif
```

**Hook de pre-commit.** Adicione ao `.pre-commit-config.yaml`:

```yaml
  - repo: https://github.com/vinicq/falsegreen
    rev: v0.6.0
    hooks:
      - id: falsegreen
```

Depois `pre-commit install`. A entrada do hook é `falsegreen --staged` com `pass_filenames: false`,
então ele lê sozinho os arquivos de teste em stage; não adicione argumentos de arquivo nem
reative `pass_filenames`, ou alguns arquivos escaneiam duas vezes. Fixe uma tag (nunca um branch)
para que execuções locais e CI usem o mesmo scanner; `pre-commit autoupdate` reescreve o `rev`.
Pule uma vez com `git commit --no-verify`, ou defina `FALSEGREEN_BLOCK=0` para o hook só avisar.
Para rodar no push em vez do commit, defina `stages: [pre-push]` sob o hook. Um git hook cru, sem
o framework:

```bash
python -m falsegreen.hook_install --repo .      # instala
python -m falsegreen.hook_install --uninstall   # remove
```

Para os casos semânticos que um parser não alcança, combine o scanner com a passagem LLM
[falsegreen-skill](skill.md).

## What it covers

O scanner mais completo da família: é a referência que os outros espelham. O detalhe completo por
código está no [catálogo Python](../catalog/python.md).

| Grupo | Códigos | Efeito |
|---|---|---|
| **Falso-positivo** (F1-F6) | ~45 códigos `C*` ativos + `CC` | ALTO bloqueia, BAIXO avisa |
| **Diagnóstico / acoplamento** (F8) | `D1`, `D3`, `D4`, `D5`, `D6`, `M2` | opcional, nunca bloqueia |
| **Projeto / CI** (F5, `--config-audit`) | `PL2` (filterwarnings não está em error), `PL7` (sem `--cov-fail-under`), `PL8` (`-x`/`--maxfail` mascara a contagem) | lê a config, reporta |

## Playwright e testes de API (C6) { #playwright-api }

O Playwright para Python cobre dois tipos de teste: E2E de UI (`page`, locators,
`expect(locator).to_be_visible()`) e testes de API (`APIRequestContext`, `assert response.ok`,
`expect(api_response).to_be_ok()`). O scanner já reconhece os dois — `expect(...)` conta como
asserção, `playwright`/`pytest_playwright` elevam o arquivo ao nível e2e/browser, e uma fixture
com `autouse=True` que só faz setup não é analisada como teste.

O refinamento que vale destacar é o `C6` (checagem fraca) sobre o idioma de API.
`assert <resp>.ok` é a forma canônica de um teste de API do Playwright asserir um 2xx (a mesma
property `.ok` existe em `requests.Response` e no `APIResponse` do Selenium). O softening de
web/UI ancorava a checagem de presença no **nome-raiz** da variável do response, então
`assert response.ok` ficava quieto mas `assert new_issue.ok` — a forma exata da doc oficial de
teste de API do Playwright — dava um falso positivo. Agora o softening ancora na **forma do
atributo** `.ok`, não no nome da variável:

- `assert new_issue.ok` / `assert issues.ok` num teste de Playwright/`requests` não dispara mais `C6`;
- o gate continua restrito ao contexto web/browser — um `assert x.ok` em nível unit ainda dispara `C6`;
- é ancorado no nó `ast.Attribute`, então um `assert ok` puro (um nome comum, guarda fraca
  legítima) e a forma de chamada `.ok()` (`.ok` é property, não é chamável) continuam disparando `C6`.

Give-back conhecido e limitado: `assert m.ok` sobre um `Mock()` sem spec num teste web também é
amaciado (o atributo auto-criado é truthy). Isso já valia para `assert response.ok`; o fix só
alarga para qualquer nome de response. Aceito sob precisão sobre recall — um falso positivo que
trava um teste de API real é pior que essa perda. Diferente do
[falsegreen-js](javascript.md#c6-weak-check), que suprime `C6` só quando a checagem fraca é o
oráculo único, o scanner Python amacia `.ok` por ocorrência; a diferença de comportamento py/js
é intencional.

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
