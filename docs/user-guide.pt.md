# Guia do usuário

Este guia passa pelas três coisas que a CLI `falsegreen-skill` faz, na ordem em que a maioria das
pessoas as encontra: revisar um teste que você já tem, escrever um teste novo que não vai te
enganar, e consertar um teste fraco. Cada seção é um passo a passo executável, não uma lista de
recursos. Copie os comandos, troque os caminhos, e você já está trabalhando.

Se você guardar só uma coisa: um teste é false-green quando fica verde enquanto o código que ele
protege está quebrado. Tudo aqui é sobre pegar isso, ou evitar que aconteça.

## Os três modos num relance

Um binário, três verbos. `analyze` julga um teste, `generate` escreve um, `fix` conserta um. Eles
compartilham o mesmo [protocolo](concepts/judgments.md) J1-J6, então um teste que o `generate`
escreve é um teste que o `analyze` aprovaria, e um patch que o `fix` propõe é um teste que precisa
sobreviver à revisão antes de você confiar.

| Verbo | Você começa com | O que faz | Contrato de saída |
|---|---|---|---|
| `analyze` | um arquivo de teste | acha smells de false-green e reporta | 0 limpo, 2 num achado ALTO (com `--fail-on-high`) |
| `generate` | uma spec com oráculo, sem teste ainda | escreve um teste e faz a autoverificação | 0 PASSED, 1 FAILED, 3 UNVERIFIED |
| `fix` | um teste fraco que o `analyze` marcou | propõe um teste mais forte e o prova com um portão de mutação | 0 aceita, 1 rejeita/não validado |

O `generate` roda o `analyze` sobre a própria saída antes de te entregar, e o `fix` prova o patch
contra um mutante antes de aceitar. Nada sai da ferramenta sem revisão.

## Antes de começar

Você precisa do Node 18 ou mais novo e de uma API key de um provedor. A CLI vem com zero
dependências npm, então não há nada para buildar.

```bash
# roda uma vez sem instalar
npx falsegreen-skill --help

# ou instala globalmente
npm install -g falsegreen-skill
```

Escolha um provedor e exporte a chave dele. O Anthropic é o padrão, então o caminho mais curto é
uma variável só:

```bash
export ANTHROPIC_API_KEY=sk-ant-...
```

Qualquer host compatível com OpenAI também funciona (Groq, OpenRouter, Ollama e outros). Veja
[escolhendo um provedor](#escolhendo-um-provedor) abaixo. Um detalhe que pega muita gente: o prompt
do protocolo tem cerca de 33k tokens, então um provedor cujo tier gratuito limita tokens por minuto
vai rejeitar a requisição. Se você levar um HTTP 413 ou 429, é por isso. Rode o Ollama local ou use
um host com tier maior.

## Modo A: revisar um teste que você já tem

Esse é o primeiro que você pega. Você tem uma suíte de testes, ela está verde, e quer saber quais
daqueles checks verdes sobreviveriam ao código dar errado.

### Passo 1 - aponte para um arquivo de teste

```bash
falsegreen-skill analyze tests/test_payment.py
```

A CLI lê o arquivo, envia com o protocolo, e imprime um relatório: cada achado carrega um código de
catálogo (como C5 ou C20), o julgamento que falhou (J1-J6), um nível de confiança e uma dica de
correção de uma linha. Testes limpos não geram achado.

### Passo 2 - leia a confiança, não só a contagem

ALTO significa que o revisor está confiante de que o teste é false-green. BAIXO significa que parece
suspeito mas pode ser intencional. Comece pelos achados ALTO: são os que vão te morder.

### Passo 3 - integre no CI

Para um pipeline, peça JSON e deixe o exit code fazer o gate:

```bash
falsegreen-skill analyze tests/test_payment.py tests/test_orders.py \
  --json --fail-on-high > report.json
```

Exit code 2 quer dizer ao menos um achado ALTO. Exit 1 quer dizer que o modelo devolveu algo que
não bateu com o schema. Exit 0 quer dizer que você está livre. Em resumo: um arquivo de teste entra,
o protocolo roda, e um achado ALTO deixa o CI vermelho para você corrigir o teste e rodar de novo;
sem achado ALTO, o CI segue verde.

## Modo B: escrever um teste que não pode mentir

Peça a qualquer modelo para "escrever um teste para esta função" e ele lê o que o código retorna
hoje e asserta isso. O teste passa, e vai continuar passando mesmo depois que um bug mudar o que o
código deveria retornar. Isso é um teste de caracterização: verde por construção, inútil como
guarda.

O Modo B se recusa a fazer isso. Ele tira o valor esperado de um oráculo que você fornece, não do
código, e depois revisa a própria saída antes de te entregar.

### Passo 1 - escreva a spec, com o oráculo

A spec é um pequeno arquivo YAML (ou JSON). Este é o exemplo que vem no projeto:

```yaml
level: unit
unit: apply_discount(price, rate)
scenario: a 15% discount on 200 returns 170
arrange:
  - price = 200
  - rate = 0.15
act: result = apply_discount(200, 0.15)
oracle:
  source: spec
  expected: "170 (200 minus 15% = 200 - 30)"
doubles: []
```

O bloco `oracle` é o ponto todo. O `expected: 170` vem da spec ("15% de desconto em 200"), não de
rodar o `apply_discount` e copiar a resposta. Tire o oráculo e o comando para antes de chamar o
modelo, porque um teste sem oráculo independente é exatamente o false-green que ele existe para
prevenir.

### Passo 2 - renderize numa linguagem

```bash
falsegreen-skill generate examples/authoring/apply-discount.spec.yaml --lang python
```

Você recebe um teste Python de verdade, com o valor esperado rastreado de volta à spec. Quer o mesmo
comportamento em TypeScript? Mesma spec, outra flag:

```bash
falsegreen-skill generate examples/authoring/apply-discount.spec.yaml --lang typescript
```

O `--lang` aceita `python`, `typescript`, `javascript`, `tsx`, `jsx` ou `robot`. `tsx` e `jsx`
cobrem o lado React da família JS/TS (o mesmo catálogo compartilhado que o `falsegreen-js` aplica
sobre `.js`/`.ts`/`.tsx`/`.jsx`). Quando você omite o `--lang`, ele assume o primeiro item de
`languages` da spec, senão `python`. Uma linguagem por execução: a spec é a fonte única, então rodar
de novo mantém as stacks equivalentes em vez de deixá-las divergir.

### Passo 3 - leia a linha da autoverificação

Depois de escrever o teste, a CLI roda o Modo A sobre ele. Três desfechos:

- **PASSED (exit 0)** - o teste gerado não disparou nenhum achado ALTO de false-green. Use.
- **FAILED (exit 1)** - o teste ainda parece false-green; a CLI já revisou uma vez e não limpou.
  Normalmente o oráculo da spec está fraco demais. Aperte e rode de novo. Esse exit também cobre
  spec ruim ou erro de API pego pelo guard offline.
- **UNVERIFIED (exit 3)** - o modelo não conseguiu produzir um relatório de revisão válido (comum em
  modelos pequenos). O teste ainda é impresso no stdout, mas não é aceito, então um pipeline nunca
  trata um teste não checado como limpo. Tente um modelo mais forte para a autoverificação.

O fluxo, em prosa: o guard offline confere se o oráculo está presente (sem oráculo, recusa com exit
1); o teste renderiza para o `--lang`; a autoverificação roda o Modo A; um achado ALTO dispara uma
revisão limitada e uma recheca; se limpar, PASSED, senão FAILED. Gate o CI no exit code, ou no
`self_check_passed` da saída `--json`.

O limite honesto: a autoverificação é uma revisão estática do mesmo modelo, não uma execução. Ela
prova que o teste não é obviamente false-green. Ela não roda o teste, não confirma que ele compila
ou importa, nem verifica o valor do oráculo. Se você disser à spec que 15% de 200 é 150, você recebe
um teste confiante, bem formado e errado. O oráculo é sua responsabilidade acertar.

### Gerar testes para a feature que você está desenvolvendo

O passo a passo do `apply_discount` é só a forma. O mesmo fluxo funciona para o que você está de
fato construindo: um serviço TypeScript, um componente React, um endpoint de API. Você descreve a
unidade e, o que mais importa, o oráculo (o resultado esperado e de onde ele vem), e a skill escreve
o teste. Há dois caminhos.

**Num host de editor (Claude Code, Cursor, Gemini).** Peça em linguagem natural. Sem arquivo de
spec:

> escreva um teste unitário para `applyPromo(cart, code)` em TypeScript; um código válido tira 10%
> do subtotal, conforme a spec de preços

O host puxa o nível e o oráculo se você deixou de fora, renderiza o teste, e roda a mesma
autoverificação. Se você nunca disser de onde vem o valor esperado, ele pergunta, porque um teste
sem oráculo independente é o false-green que ele se recusa a escrever.

**Na CLI.** Coloque as mesmas respostas numa spec e escolha a stack. Uma spec de feature realista:

```yaml
level: unit
unit: applyPromo(cart, code)
scenario: SAVE10 takes 10% off a 200 subtotal, per the pricing spec
languages: [TypeScript]
arrange:
  - cart = { subtotal: 200 }
  - code = "SAVE10"
act: result = applyPromo(cart, "SAVE10")
oracle:
  source: spec
  expected: "180 (200 minus 10% = 200 - 20), from the pricing spec"
doubles: []
```

```bash
falsegreen-skill generate promo.spec.yaml --lang typescript
```

Você recebe um teste TypeScript de verdade, cujo `180` esperado se rastreia de volta à spec de
preços, não ao que o `applyPromo` por acaso retorna hoje.

O lado React da família funciona do mesmo jeito. Uma spec de componente renderiza via Testing
Library:

```bash
falsegreen-skill generate profile-card.spec.yaml --lang tsx
```

O teste renderizado importa seu framework de forma explícita e asserta contra o estado visível
(`screen.getByRole(...)`), não contra o valor de retorno da chamada de render, e passa na
autoverificação como qualquer outro. `--lang jsx` faz o mesmo para React em JS puro.

Esse último ponto é a regra para qualquer coisa acima de uma função pura: o oráculo de um componente
ou de um teste ponta a ponta é o estado visível que um usuário veria, não a saída interna do render.
É a [pirâmide](concepts/pyramid.md) de novo: quanto mais alto o nível, mais o oráculo é "o que o
usuário observa".

## Modo C: consertar um teste fraco e provar o conserto

O `analyze` achou um false-green. O `fix` propõe uma versão mais forte e então a prova antes de você
confiar. É opcional, só Python e pytest por enquanto, e nunca toca no seu código de produção nem
aplica o patch sozinho.

### Passo 1 - nomeie o achado e o código que ele protege

```bash
falsegreen-skill fix tests/test_discount.py --case C2b --line 14 \
  --sut src/discount.py --sut-line 12
```

`--case` e `--line` vêm direto do relatório do `analyze`. `--sut` é o arquivo de produção que o
teste deveria proteger, e `--sut-line` é a linha de comportamento que o achado aponta (o padrão é o
`--line`). O conjunto corrigível da V1 é `C2b`, `C20`, `C21`, `C5` e `C7`.

### Passo 2 - confie no portão, não no modelo

A CLI monta uma cópia limpa do seu código e roda três checagens no patch proposto: ele faz parse,
passa no pytest contra o código real, e falha quando um único operador na linha do SUT é invertido.
Um patch só é aceito quando passa no código correto e fica vermelho no mutante. Essa última checagem
é o que separa uma asserção de verdade de uma nova tautologia.

Sem `--sut` (ou com `--cheap`) o portão não tem o que mutar, então cai para só-propor e diz que o
patch não foi validado. O exit code é 0 quando aceita, 1 quando rejeita ou não valida, então o CI
pode ramificar nele.

O limite honesto: o portão prova que o patch pega aquele mutante, não todo bug que possa existir. É
um piso, não uma garantia. JS/TS/Robot e os casos semânticos profundos (10/11/12/18) são v2.

## Escolhendo um provedor

Os três provedores embutidos leem cada um a própria chave e não precisam de `--base-url`; trocar de
provedor é uma flag:

```bash
# Claude (Anthropic) - o provedor padrão, nada extra a passar
export ANTHROPIC_API_KEY=sk-ant-...
falsegreen-skill analyze tests/test_payment.py
falsegreen-skill generate promo.spec.yaml --lang typescript

# Codex (OpenAI) - modelos GPT / série o
export OPENAI_API_KEY=sk-...
falsegreen-skill analyze tests/test_payment.py --provider openai --model gpt-5

# Gemini (Google)
export GEMINI_API_KEY=...
falsegreen-skill analyze tests/test_payment.py --provider gemini
```

Qualquer outro host compatível com OpenAI funciona via `--provider openai-compatible`: aponte
`--base-url` para a raiz `/v1` e passe o id do modelo.

```bash
export FALSEGREEN_API_KEY=sk-or-...
falsegreen-skill analyze tests/test_payment.py \
  --provider openai-compatible \
  --base-url https://openrouter.ai/api/v1 \
  --model meta-llama/llama-3.3-70b-instruct --max-tokens 8192
```

A tabela completa de provedores (URLs base, modelos de exemplo e quais tiers gratuitos cabem no
prompt de 33k) está na [página da skill](scanners/skill.md#openai-compatible) e, com mais
profundidade, no [cli.md](https://github.com/vinicq/falsegreen-skill/blob/master/docs/cli.md) do
projeto. Um guia rápido:

| Quer | Use |
|---|---|
| Setup mais simples, melhor profundidade | `anthropic` (padrão), `--model claude-opus-4-8` para o caso 18 mais difícil |
| Sem chave, sem custo | Ollama local (`--base-url http://localhost:11434/v1`), qualquer chave placeholder |
| Um tier hospedado compatível com OpenAI | OpenRouter ou NVIDIA NIM cabem no prompt de 33k na maioria dos modelos |
| Rodar sem chave própria nenhuma | instale a skill como plugin de editor e deixe o modelo do host trabalhar |

O `generate` e a autoverificação com `--json` precisam de um modelo que caiba no prompt de ~33k
tokens e consiga emitir o relatório JSON. Um modelo local minúsculo costuma cair em UNVERIFIED por
isso.

## Quando algo dá errado

| Sintoma | Causa | O que fazer |
|---|---|---|
| HTTP 413 ou 429 logo de cara | O prompt de 33k passa do limite gratuito de tokens por minuto do provedor | Use um tier maior, um modelo de contexto grande, ou o Ollama local |
| `generate` diz UNVERIFIED | O modelo não conseguiu emitir um relatório de revisão válido | Use um modelo mais forte para a autoverificação |
| Saída JSON "could not parse" | Um modelo de raciocínio gastou o orçamento pensando e foi cortado | Suba o `--max-tokens` para 8192 ou mais |
| `generate` recusa antes de qualquer chamada | A spec não tem `oracle.expected`, ou o `--lang` é desconhecido | Adicione o bloco de oráculo, ou corrija a flag de linguagem |
| `fix` diz "unvalidated" | Você não passou `--sut` / `--sut-line` | Passe os dois para o portão de mutação poder rodar |

## Para onde ir depois

- [falsegreen-skill (LLM)](scanners/skill.md) - a referência da CLI neste site: cada flag, a tabela
  completa de provedores e os passos de habilitação por host
- [Julgamentos (J1-J6)](concepts/judgments.md) - o protocolo que todos os modos compartilham
- [A hierarquia de oráculos](concepts/oracle.md) - por que o valor esperado não pode vir do código
- [Catálogo semântico](catalog/semantic.md) - os códigos que a skill reporta
