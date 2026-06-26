# Fundamentação de pesquisa

O falsegreen é tanto um projeto de pesquisa quanto uma ferramenta. Tem um propósito duplo:
**acadêmico** - uma taxonomia defensável, um denominador com nome, ameaças à validade - e
**industrial** - poucos falsos positivos, padrões reais, algo que roda em CI. Cada código do
catálogo remonta a um modo de falha e a um julgamento, então a afirmação por trás dele é
verificável, não folclore.

## A metodologia (nossa base)

A abordagem inteira repousa sobre quatro pilares, cada um com sua própria página:

- **[Taxonomia de falhas F1-F8](concepts/taxonomy.md)** - o eixo conceitual: *como* um teste
  passa em verde sem proteger nada, independente de linguagem.
- **[Julgamentos J1-J6](concepts/judgments.md)** - seis perguntas feitas a um único teste; um
  achado nomeia a garantia exata que falha, não um cheiro vago.
- **[A hierarquia de oráculos](concepts/oracle.md)** - o valor esperado precisa vir de uma
  fonte independente do código; promover o próprio código a oráculo é como um bug congela como
  "correto".
- **[O portão de correção por IA (F7)](scanners/skill.md)** - um portão de mutação
  bidirecional: um teste reforçado precisa passar no código limpo e falhar no bug
  reintroduzido, ou é rejeitado.

## O denominador e as ameaças à validade

Precisão e revocação são reportadas contra um universo com nome, não contra uma lista aberta.
A família mede contra o [Open Catalog of Test Smells](https://test-smell-catalog.readthedocs.io/)
(517 smells documentados, 1621 referências, 166 fontes), e só a fatia false-green está no
escopo. O que fica de fora e por quê está na página [cobertura vs a literatura](concepts/denominator.md) - essa
página *é* a declaração de ameaças à validade em forma pública.

## Linhas de base da literatura

Para contexto de comparação, os detectores e estudos publicados no espaço adjacente:

| Ferramenta / estudo | Precisão | Revocação | F1 | Escopo |
|---|---|---|---|---|
| xNose (Paul, 2024) | 96.97% | 96.03% | - | C#, 16 smells |
| srcML (Lopes, 2023) | 87.25% | 100% | - | C++ and Java, 7 smells |
| JNose (Goes, 2024) | 85-100% | 90-100% | - | Java, 6 smells |
| LLM CoT + one-shot (Santana, 2025) | - | - | 0.732 Py / 0.763 Java | Python and Java |

Nossa própria avaliação contra esse denominador vive no hub de pesquisa; os números são
liberados quando são publicados, não antes.

## O estudo

O código do produto e esta documentação são públicos. O dataset, a adjudicação smell a smell,
e os resultados não publicados vivem num **hub de pesquisa privado**, então nenhum número ou
evidência não publicada aparece num repositório público. Resultados e qualquer artigo são
linkados aqui quando publicados.

Materiais públicos do estudo:

- [falsegreen](https://github.com/vinicq/falsegreen) (Python), [falsegreen-js](https://github.com/vinicq/falsegreen-js)
  (JS/TS), [robotframework-falsegreen](https://github.com/vinicq/robotframework-falsegreen)
  (Robot), [falsegreen-skill](https://github.com/vinicq/falsegreen-skill) (semântico).
- O trabalho fundador e a lista completa de referências: [créditos e referências](credits.md).
- O denominador da literatura: [Open Catalog of Test Smells](https://test-smell-catalog.readthedocs.io/).

## Como citar

Se você usa o falsegreen em trabalho acadêmico, cite o repositório de produto relevante e a
literatura fundadora de rotten-green-test listada em [créditos](credits.md). Uma entrada de
citação canônica é adicionada aqui assim que o estudo for publicado.
