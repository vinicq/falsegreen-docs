# falsegreen-skill (passagem semântica com LLM)

[![CI](https://github.com/vinicq/falsegreen-skill/actions/workflows/ci.yml/badge.svg)](https://github.com/vinicq/falsegreen-skill/actions/workflows/ci.yml)
[![npm](https://img.shields.io/npm/v/falsegreen-skill.svg)](https://www.npmjs.com/package/falsegreen-skill)
[![Downloads](https://img.shields.io/npm/dm/falsegreen-skill.svg)](https://www.npmjs.com/package/falsegreen-skill)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/vinicq/falsegreen-skill/blob/master/LICENSE)

A camada semântica e um superconjunto dos três scanners estáticos. Ela lê um teste contra a
intenção, a spec e o código de produção para pegar os false-greens que nenhum parser vê (F4, F7),
e carrega todo código estrutural dos scanners mais a série S, exclusiva de IA.

- Repositório: [github.com/vinicq/falsegreen-skill](https://github.com/vinicq/falsegreen-skill)
- Catálogo: [códigos semânticos](../catalog/semantic.md)

## What it is

Não é específica do Claude. O mesmo protocolo vem empacotado para Claude Code, Codex, Gemini,
Cursor, prompts de LLM avulsos, uso via API e uma CLI npm. Para Python ela aplica o catálogo
completo do falsegreen diretamente; para TypeScript, JavaScript e Robot Framework ela é a
ferramenta de detecção primária.

## The protocol (J1-J6)

Todo teste é lido por seis [julgamentos](../concepts/judgments.md): a asserção roda, o valor
esperado vem de um [oráculo independente](../concepts/oracle.md), a unidade real é exercitada, a
asserção é suficiente, está livre de acoplamento com internos, passa em isolamento. Um falso
positivo é pior que um escape, então um achado de valor-errado não é reportado sem citar um
oráculo.

## Install and first run

A skill roda em vários hosts (Claude Code, Codex, Gemini, Cursor) e como CLI npm
avulsa. A CLI precisa só de Node 18+ e de uma API key do provedor que você
escolher:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
npx falsegreen-skill analyze tests/test_demo.py
```

`--provider openai` ou `--provider gemini` troca o modelo; `--json` e
`--fail-on-high` integram em CI. A configuração por host (o plugin do Claude
Code, as extensões de Codex e Gemini, a regra do Cursor) está no
[README da skill](https://github.com/vinicq/falsegreen-skill#installation).

## First finding

Dado um teste que asserta o mock de volta para ele mesmo:

```python
# tests/test_tax.py
def test_calculate_tax(mock_calc):
    mock_calc.return_value = 0.15
    result = calculate_tax(100, mock_calc)
    assert result == mock_calc.return_value
```

`npx falsegreen-skill analyze tests/test_tax.py` reporta:

```
CASE 11 (J2) - HIGH - Python - unit - behavior

Test: test_calculate_tax (line 2-5)
Finding: The assertion checks mock_calc.return_value - the same value the mock
was configured to return. It passes for any result, including a wrong one.
Evidence:
  mock_calc.return_value = 0.15
  assert result == mock_calc.return_value
Fix hint: Assert against an independently computed value, e.g. assert result == 15.0.
```

## Reading a finding

Cada achado nomeia cinco coisas:

- `CASE 11` - o [código semântico](../catalog/semantic.md). A skill também emite
  os códigos estruturais `C*`, `JS*` e `R*` dos scanners.
- `(J2)` - o [julgamento](../concepts/judgments.md) que falhou: o valor esperado
  não veio de um [oráculo](../concepts/oracle.md) independente.
- `HIGH` - confiança. HIGH é quando não há leitura legítima plausível; LOW avisa.
- `Python - unit - behavior` - linguagem, [nível na pirâmide](../concepts/pyramid.md)
  e intenção do teste.
- `Finding` / `Evidence` / `Fix hint` - o que está errado, as linhas que provam, e
  como corrigir.

## What it covers

A ferramenta mais ampla da família. É o superconjunto, então sua cobertura é a união de tudo que
os scanners pegam mais o que só um leitor de intenção consegue:

| Camada | Cobertura |
|---|---|
| **Todos os códigos estruturais** | todo código `C*` (Python), `JS*` e `R*` dos três scanners, aplicado pela leitura do código-fonte |
| **Casos semânticos** | 10 (mocka a unidade), 11 (asserta o stub), 12 (reimplementa a fórmula), 15 (estado compartilhado), 18 (contradiz a spec) |
| **A série S** (exclusiva de IA) | `S1`-`S13`: intenção divergente, oráculo irrelevante, valor plausível mas errado, oráculo grosseiro, testa o framework, só caminho feliz, valor tirado da saída, mock por indireção, arranjo autorrealizável, asserta o log, checagem de segurança só negativa, faz patch na lógica central, dependência de ordem entre arquivos |
| **Passagens em DSL** | Gherkin `.feature` e Tavern `*.tavern.yaml` (veja [Gherkin e Tavern](../catalog/gherkin-tavern.md)) |
| **Consciência de nível** | lê unit / integration / E2E a partir de sinais e ajusta o oráculo |

## Modes

- **Detect** - lê uma suíte e reporta achados (J1-J6, nível, evidência, dica de correção).
- **Author** - gera testes que não são false-green por construção, uma spec por nível da pirâmide.
- **AI-fix gate (F7)** - propõe um teste reforçado e o valida com um portão de mutação
  bidirecional (passa em código limpo, falha com o bug reintroduzido).

## What it does not cover, and why

### A metade de runtime do F7
A skill propõe um teste reforçado e autoverifica que ele *consegue* falhar, mas não roda teste de
mutação: esse é o trabalho do host (mutmut, cosmic-ray, Stryker). Um teste reforçado só é *aceito*
depois que o portão bidirecional roda, e a skill nunca invoca a ferramenta de mutação. Então o
resultado do portão ao vivo fica fora das mãos da skill, por design.

### O eixo errado
Mesmo sendo a ferramenta mais ampla, ela continua só de false-green. Fragilidade/falso-vermelho,
higiene pura, lentidão, design, nomenclatura e smells de cultura-de-runtime ficam de fora, o mesmo
limite dos scanners. Veja [cobertura vs a literatura](../concepts/denominator.md).

### Trade-off de determinismo
Os achados semânticos são confirmados pelo operador, não determinísticos: a confiança é BAIXO ou
ALTO conforme o quão clara é a contradição, e a skill mostra seu raciocínio em vez de bloquear
automaticamente. Onde um parser consegue provar um padrão, os [scanners estáticos](python.md) são
o pré-filtro mais rápido e determinístico; a skill é a rede completa multi-stack. Veja
[escopo e honestidade](../concepts/scope.md).
