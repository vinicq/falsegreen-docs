# Pirâmide de testes e níveis

Os scanners descobrem um arquivo de teste de forma agnóstica de nível, mas alguns códigos são
lidos de modo diferente conforme o nível. Saber o nível evita um falso positivo: o que é uma
assertion fraca num teste de unidade é a assertion correta num teste ponta-a-ponta.

| Nível | Oráculo | Como é a cobertura real |
|---|---|---|
| **Unidade** | doubles nas fronteiras; `assert` / `expect` no valor de retorno | a lógica da própria unidade, em isolamento |
| **Integração (API + BD)** | a resposta ou linha real é a verificação | um cliente ou store ao vivo, sem double |
| **E2E** | um elemento de página ou estado está presente | um driver de navegador ou dispositivo |

## O nível é lido por sinais, não chutado

As ferramentas leem o nível a partir de pistas concretas, com uma precedência fixa (o sinal mais
forte vence):

1. Uma **fronteira com double** mantém o teste em unidade/componente mesmo que um cliente real
   seja importado. O mock *é* a fronteira.
2. Senão, uma **fronteira real** (um cliente HTTP ao vivo, um ORM, um driver de banco) torna
   integração.
3. Senão, um **driver de navegador ou dispositivo** torna E2E.
4. Nenhum sinal significa unidade.

Um bloco `conventions:` sobrescreve tudo isso. Marcadores como `smoke`, `slow`, `asyncio` ou
`anyio` são neutros de nível.

## As isenções cientes do nível

Alguns códigos se seguram uma vez que o nível está claro, para não dispararem no padrão correto:

- **`C6`** (verificação fraca de truthiness) não dispara em `assert response` num teste HTTP ou
  `assert locator` num teste Playwright. Nessas camadas, presença *é* o contrato.
- **`C14`** (golden file a partir da saída) não dispara em snapshot testing de navegador, onde
  uma baseline gerada é o fluxo pretendido.
- **`C16`** (tempo descontrolado) não dispara quando `freezegun` ou `time-machine` é importado,
  porque o relógio está com double.

## A regra-mãe

Uma chamada real a banco ou API dentro de um teste que se diz de unidade é, por si só, o
smell. Não é questão de categoria: é mystery guest, resource optimism, o inverso do
over-mocking. Um banco é integração quando bate num datastore real, mesmo um SQLite em memória
ou um testcontainer; é unidade quando a camada de dados está com double.

Os proxies estáticos para "I/O real num teste de unidade" são `C23` (caminho ou URL de IP
hard-coded, nos três scanners) e `C29` / `C30` em Python. A [skill](../scanners/skill.md) lê o
nível diretamente e relaxa ou aperta o oráculo conforme.
