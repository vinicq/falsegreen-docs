# Cobertura por família de falha

A [página do denominador](denominator.md) nomeia o universo contra o qual a família se mede (o
[Open Catalog of Test Smells](https://test-smell-catalog.readthedocs.io/), 517 smells documentados)
e o eixo em escopo (só false-green). Esta página desce um nível: por família de falha
([F1-F8](taxonomy.md)), quais códigos os quatro scanners de fato entregam, com link para o código
público que comprova.

## O que esta página conta, e o que não conta

Esta é uma visão de **código público**, não uma avaliação. Ela conta códigos que o ecossistema
entrega por família, cada um com link para a entrada do catálogo ou o scanner que o emite. Ela
**não** reporta precisão ou recall contra o catálogo, e não carrega evidência de dataset. Esses
números são medidos contra a fatia false-green do denominador e saem com o estudo, não aqui. A
leitura honesta da tabela abaixo: "a família entrega estes códigos para este modo de falha", não "a
família detecta N% da literatura".

Um código pode aparecer em mais de um scanner quando o mesmo id cobre o mesmo mecanismo em outra
linguagem (C5 é a assertion sempre-verdadeira em Python, JS/TS e Robot). Contar uma família por ids
distintos, não por linhas de scanner, evita contagem dupla.

## Os quatro scanners

| Scanner | Linguagem | Catálogo público (a lista de códigos) |
|---|---|---|
| [falsegreen](https://github.com/vinicq/falsegreen) | Python / pytest | [catálogo no README](https://github.com/vinicq/falsegreen#what-it-detects) · [`scanner.py`](https://github.com/vinicq/falsegreen/blob/main/src/falsegreen/scanner.py) |
| [falsegreen-js](https://github.com/vinicq/falsegreen-js) | JS / TS | [catálogo no README](https://github.com/vinicq/falsegreen-js#readme) |
| [robotframework-falsegreen](https://github.com/vinicq/robotframework-falsegreen) | Robot Framework | [catálogo no README](https://github.com/vinicq/robotframework-falsegreen#readme) |
| [falsegreen-skill](https://github.com/vinicq/falsegreen-skill) | semântico (LLM) | [`reference.md`](https://github.com/vinicq/falsegreen-skill/blob/master/reference.md) (o superset) |

A skill é o superset: todo código estrutural que os três scanners estáticos emitem aparece no seu
[`reference.md`](https://github.com/vinicq/falsegreen-skill/blob/master/reference.md), mais os códigos
só-semânticos (casos 10, 11, 12, 15, 18) que nenhum parser alcança.

## Códigos por família

A taxonomia ([F1-F8](taxonomy.md)) é o eixo conceitual: como um teste passa verde sem proteger nada.
Os códigos abaixo são os ids públicos que o ecossistema entrega para cada modo. Scanners estáticos
cobrem F1-F3, F5 e os proxies estáticos de F6; a skill adiciona F4 e F7; F8 é o grupo de
diagnóstico, desligado por padrão.

| Família | Modo de falha | Códigos que o ecossistema entrega | Camada |
|---|---|---|---|
| **F1** | Não checa nada (sem oráculo) | `C2`, `C2b`, `C2c`, `C27`, `C39`, `C50`, `C51`, `JS2`, `JS6`, `JS13`, `R2`, `R4`, `R7`, casos semânticos 10/11 | estático + skill |
| **F2** | A checagem existe mas nunca roda | `C1`, `C3`, `C20`, `C21`, `C22`, `C43`, `CC`, `JS5`, `JS7`, `JS9`, `JS11`, `JS25`, `JS26`, `JS29`, `JS31`, `R8`, `R8b` | estático |
| **F3** | A checagem é trivial (sempre passa) | `C5`, `C6`, `C6c`, `C7`, `C8`, `C8b`, `C11a`, `C18`, `C34`, `C42`, `C44`, `C52`, `JS15`, `JS21`, `JS30`, `R1`, `R6` | estático |
| **F4** | Checa a coisa errada | `C9`, `C9b`, `C19`, `C28`, `C49`, `C55`, `JS8`, `JS24`, `JS27`; caso semântico 18, partes de `C6` / `C33` / os códigos de snapshot | estático (parcial) + skill |
| **F5** | Sai da contagem (skip / não coletado) | `C4`, `C4b`, `C25`, `C32`, `C38`, `C45`, `JS1`, `JS4`, `JS22`, `JS23`, `R3`, `R5`; camada de projeto: `PL1`, `PL2`, `PL7`, `PL8`, `PL9`, `PL10` | estático + camada de projeto |
| **F6** | Passa ou falha por sorte (não-determinismo) | `C16`, `C23`, `C24`, `C29`, `C35` (proxies estáticos) | estático (proxy) + runtime |
| **F7** | Oráculo circular ou semântico | casos semânticos 10, 11, 12, 15; `C14` (o canto codificável) | skill + mutation testing |
| **F8** | Higiene / legibilidade (não é false-green) | `D1`, `D3`, `D4`, `D5`, `D6`, `D7`, `D8`, `M2` (diagnósticos opcionais) | diagnóstico / linter |

A lista exata e atual de códigos por scanner vive no catálogo do README de cada repositório e no
[`reference.md`](https://github.com/vinicq/falsegreen-skill/blob/master/reference.md) da skill. Esta
tabela agrupa esses códigos publicados por modo de falha; ela não inventa novos. Onde um código
mapeia para mais de uma família (um código pode ser ao mesmo tempo "nunca roda" e "fraco"), ele é
listado sob a família que nomeia seu mecanismo principal.

## O que é e o que não é contado por família

- **F1, F2, F3** são totalmente estáticos e saturados: um parser por arquivo os prova sem falsos
  negativos dentro das regras. Os READMEs dos scanners listam cada id.
- **F4** é contado só para a fatia que um parser alcança (uma comparação de formato de string, uma
  métrica descartada). O núcleo "contradiz a spec" é semântico e vive na skill (caso 18); não é uma
  contagem estática.
- **F5** tem duas fatias: a fatia por arquivo (um teste não coletado, um xfail não-strict) contada
  nos códigos dos scanners, e a fatia de projeto (`PL1`, `PL2`, `PL7`, `PL8`, `PL9`, `PL10`, lida pelo `--config-audit`) contada à
  parte. A fatia de runtime (um erro de coleta reportado como "0 testes") é documentada, não um
  código.
- **F6** é contado só como proxies estáticos (`C16` para tempo/aleatoriedade descontrolados, `C23`
  para caminho hard-coded). Se um teste é flaky na prática precisa de runtime e fica fora de banda,
  então não é contado aqui.
- **F7** é a família semântica. Só `C14` (um snapshot gerado da própria saída do código) é um código
  estático; o resto (mockar a unidade sob teste, reimplementar a fórmula, estado emprestado) são
  casos da skill e se confirmam com mutation testing, que a skill nunca roda. Eles são listados, não
  contados como cobertura estática.
- **F8** não é false-green. Os códigos de diagnóstico ficam desligados por padrão, e linters
  dedicados (ruff, ESLint, Robocop) cobrem o mesmo terreno. Aparecem sob demanda, não são prometidos
  como detecção.

## Por que esta é a moldura honesta

Uma afirmação de cobertura só tem sentido contra um denominador nomeado. O percentual do catálogo
que a família detecta é reportado contra a [fatia false-green](denominator.md), não contra os 517
inteiros, e esse número sai com o estudo. Esta página para no que o código público prova: quais
códigos existem por família, onde está o código, e qual camada cuida de cada modo. A fronteira na
[página de escopo](scope.md) e o cruzamento na [página do denominador](denominator.md) dizem o que
fica de fora e por quê.
