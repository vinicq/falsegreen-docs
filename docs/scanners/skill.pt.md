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

## Complete usage and configuration { #complete-usage-and-configuration }

A primeira execução acima é o caminho de cinco minutos. Esta seção é a referência completa: a CLI
(`analyze` e o modo de portão de mutação `fix`), a configuração de provedor para cada backend
suportado e os passos de habilitação por host. Espelha o que o [README do projeto](https://github.com/vinicq/falsegreen-skill),
o [docs/cli.md](https://github.com/vinicq/falsegreen-skill/blob/master/docs/cli.md) e o
[providers.md](https://github.com/vinicq/falsegreen-skill/blob/master/providers.md) documentam.

### The CLI { #the-cli }

Node 18 ou mais novo, zero dependências. Instale ou rode sob demanda:

```bash
npm install -g falsegreen-skill
npx falsegreen-skill analyze tests/test_payment.py
```

Dois comandos:

```text
falsegreen-skill analyze <file...> [options]
falsegreen-skill fix <test-file> --case <code> --line <n> [options]
```

A CLI envia cada arquivo a um provedor de LLM com o protocolo J1-J6 como system prompt e imprime o
relatório de achados. Ela identifica a linguagem pela extensão do arquivo, então Python, TypeScript
e JavaScript funcionam do mesmo jeito, sem flag extra.

#### `analyze` flags { #analyze-flags }

```bash
npx falsegreen-skill analyze tests/test_orders.py                 # um arquivo
npx falsegreen-skill analyze tests/test_orders.py tests/test_pay.py  # vários (chamadas separadas)
npx falsegreen-skill analyze tests/ --json --fail-on-high          # gate de CI: sai com 2 num achado ALTO
npx falsegreen-skill analyze tests/ --model claude-opus-4-8        # modelo mais profundo para caso 18
npx falsegreen-skill analyze tests/ --temperature 0.0              # mais determinístico (padrão 0.2)
```

| Flag | Descrição | Padrão |
|---|---|---|
| `--provider <name>` | `anthropic`, `openai`, `gemini` ou `openai-compatible` | `anthropic` |
| `--model <model>` | Override de modelo. Obrigatório para `openai-compatible` | por provedor (abaixo) |
| `--base-url <url>` | URL base da API. Obrigatória para `openai-compatible` | nenhuma |
| `--json` | Valida e emite JSON conforme `schema/report.json` | off |
| `--conventions <file>` | Bloco YAML/texto de convenções injetado conforme o Passo 0 do SKILL.md | nenhum |
| `--temperature <n>` | Temperatura de amostragem 0.0-1.0. Ignorada para a série o da OpenAI (o3, o4-mini) | `0.2` |
| `--max-tokens <n>` | Máximo de tokens de saída por requisição | `4096` |
| `--fail-on-high` | Sai com 2 quando há algum achado ALTO. Requer `--json` | off |

Com `--json`, cada resposta do modelo é validada contra o schema canônico e emitida como um
relatório agregado. Se o projeto usa helpers de asserção próprios ou padrões intencionais que
parecem smells, declare-os num arquivo de convenções e passe `--conventions`; o bloco é injetado
antes do conteúdo do arquivo, conforme o Passo 0 do protocolo, então o modelo o lê antes de julgar.

#### `fix` mode (portão de mutação) { #fix-mode }

`analyze` acha um false-green; `fix` propõe um teste mais forte e o prova antes de você confiar.
É opcional, só Python/pytest, e apenas propõe: imprime um patch do arquivo de teste mas nunca o
aplica e nunca edita o código de produção.

```bash
# propõe um patch para um achado C2b e roda o portão contra o SUT real
npx falsegreen-skill fix tests/test_discount.py --case C2b --line 14 --sut src/discount.py

# só parse + preserve, sem portão de mutação (sem SUT executável, ou passe rápido)
npx falsegreen-skill fix tests/test_discount.py --case C5 --line 9 --cheap

# veredito do portão legível por máquina (schema/fix-validation.json)
npx falsegreen-skill fix tests/test_discount.py --case C20 --line 22 --sut src/discount.py --json
```

| Flag | Descrição |
|---|---|
| `--case <code>` | Código de catálogo do achado a corrigir (`C2b`, `C20`, `C21`, `C5`, `C7`) |
| `--line <n>` | Linha do achado no arquivo de teste (1-indexada) |
| `--sut <file>` | Arquivo de produção que o teste protege. Obrigatório para um fix validado; sem ele o portão cai para só-propor |
| `--sut-line <n>` | Linha no SUT a mutar (a linha de comportamento que o achado aponta). Padrão é `--line` |
| `--cheap` | Tier de validação: só parse + preserve, sem portão de mutação. O tier padrão é forte (parse + preserve + mutação) |

O portão roda três checagens numa réplica limpa: o patch faz parse, passa no `pytest` contra o
código real, e **falha** numa mutação de linha do SUT. Um patch só é aceito quando passa no código
correto e fica vermelho no mutante, o que prova que a nova asserção pega um bug em vez de ser uma
nova tautologia. Sem `--sut` ele cai para só-propor e diz que o fix não foi validado. O limite
honesto: o portão prova que o fix pega o mutante alvo, não todo bug possível.

#### Exit codes { #cli-exit-codes }

| Código | Significado |
|---|---|
| `0` | Análise concluída (ainda pode haver achados; `analyze` é ferramenta de análise, não gate) |
| `1` | Erro: arquivo ausente, chave de API ausente, flag inválida, JSON inválido, schema divergente, resposta de API fora de 2xx |
| `2` | `--fail-on-high` foi setado e o relatório JSON tem ao menos um achado ALTO |

### Provider configuration { #provider-configuration }

A skill não está presa ao Claude. O protocolo é agnóstico de provedor; escolha um backend com
`--provider` e defina a chave de API correspondente no ambiente.

| Variável | Usada por |
|---|---|
| `ANTHROPIC_API_KEY` | `--provider anthropic` |
| `OPENAI_API_KEY` | `--provider openai` (e fallback para `openai-compatible`) |
| `GEMINI_API_KEY` | `--provider gemini` |
| `FALSEGREEN_API_KEY` | `--provider openai-compatible` (tem precedência sobre `OPENAI_API_KEY`) |

Modelos padrão: `anthropic` usa `claude-sonnet-4-6`, `openai` usa `gpt-4o`, `gemini` usa
`gemini-2.5-pro`. Para análise profunda do caso 18, passe `--model claude-opus-4-8` (Anthropic) ou
`--model o3` (OpenAI); ao usar `o3`, `--temperature` é ignorado automaticamente.

```bash
# Anthropic (padrão)
export ANTHROPIC_API_KEY=sk-ant-...
falsegreen-skill analyze tests/test_payment.py

# OpenAI
export OPENAI_API_KEY=sk-...
falsegreen-skill analyze tests/test_payment.py --provider openai

# Google Gemini
export GEMINI_API_KEY=...
falsegreen-skill analyze tests/test_payment.py --provider gemini
```

#### openai-compatible (base-url) { #openai-compatible }

Qualquer endpoint compatível com OpenAI funciona via `--provider openai-compatible` com um
`--base-url` e um `--model` explícitos. Defina `FALSEGREEN_API_KEY` com a chave daquele provedor.

```bash
# Groq (LLaMA rápido)
export FALSEGREEN_API_KEY=gsk_...
falsegreen-skill analyze tests/test_payment.py \
  --provider openai-compatible \
  --base-url https://api.groq.com/openai/v1 \
  --model llama-3.3-70b-versatile

# Ollama (local)
export FALSEGREEN_API_KEY=ollama
falsegreen-skill analyze tests/test_payment.py \
  --provider openai-compatible \
  --base-url http://localhost:11434/v1 \
  --model qwen2.5-coder:32b
```

Modelos de raciocínio em hosts openai-compatible usam o mesmo formato, só um id de modelo mais
forte:

```bash
# Nvidia NIM (raciocínio DeepSeek R1)
export FALSEGREEN_API_KEY=nvapi-...
falsegreen-skill analyze tests/test_orders.py \
  --provider openai-compatible \
  --base-url https://integrate.api.nvidia.com/v1 \
  --model deepseek-ai/deepseek-r1

# Fireworks (raciocínio DeepSeek R1)
export FALSEGREEN_API_KEY=fw_...
falsegreen-skill analyze tests/test_orders.py \
  --provider openai-compatible \
  --base-url https://api.fireworks.ai/inference/v1 \
  --model accounts/fireworks/models/deepseek-r1

# Groq (DeepSeek R1 distill, raciocínio)
export FALSEGREEN_API_KEY=gsk_...
falsegreen-skill analyze tests/test_orders.py \
  --provider openai-compatible \
  --base-url https://api.groq.com/openai/v1 \
  --model deepseek-r1-distill-llama-70b
```

Para os achados mais difíceis do [caso 18](../catalog/semantic.md) (valor esperado contradiz a
spec), o padrão mantido é uma chamada de duas passagens finder/refuter com um modelo de raciocínio,
documentada no `providers.md` do projeto.

### Per-host enable { #per-host-enable }

O mesmo protocolo é empacotado para cada host. Escolha o caminho que casa com seu editor ou agente.

**Claude Code (caminho principal).** Adicione o marketplace e instale o plugin:

```text
/plugin marketplace add vinicq/falsegreen-skill
/plugin install falsegreen-skill@falsegreen
```

Depois invoque `/falsegreen-skill:falsegreen-llm`, ou anexe um arquivo de teste e peça a análise de
falso-positivo, a skill dispara por intenção. Para Python ela aplica o catálogo completo de padrões
direto; opcionalmente rode antes o [scanner Python](python.md) estático e passe a saída dele à skill
como a passagem estrutural.

**OpenAI Codex CLI.** Dois caminhos: o marketplace de plugins
(`codex plugin marketplace add vinicq/falsegreen-skill`), ou clonar o repo, onde o `AGENTS.md` da
raiz carrega automaticamente quando o Codex abre uma sessão dentro do clone.

**Gemini CLI.** Instale a extensão:

```bash
gemini extensions install https://github.com/vinicq/falsegreen-skill
```

O manifesto carrega o `GEMINI.md` como contexto persistente, então toda sessão carrega o protocolo
J1-J6; pergunte em linguagem natural. Uma skill de workspace em
`.gemini/skills/falsegreen-skill/SKILL.md` é a alternativa quando você quer a descoberta de skills
do Gemini em vez do contexto da extensão inteira.

**Cursor.** Copie o template da regra para `.cursor/rules/falsegreen-skill.mdc` (template completo
no `contexts/cursor.md` do projeto). O Cursor o carrega num arquivo de teste que casa; peça ao
Cursor para analisar o arquivo e o protocolo J1-J6 roda.

**LLM puro / API.** Use os snippets de SDK no
[providers.md](https://github.com/vinicq/falsegreen-skill/blob/master/providers.md) do projeto: o
system prompt é o `SKILL.md`, a mensagem do usuário é o arquivo de teste, e a saída JSON estruturada
segue o `schema/report.json`. O mesmo padrão de base-url da CLI cobre qualquer backend compatível
com OpenAI.

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
