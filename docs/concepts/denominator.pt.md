# Cobertura versus a literatura

A família se mede contra o [Open Catalog of Test Smells](https://test-smell-catalog.readthedocs.io/)
(UFAL / easy-software): 517 smells documentados em 1621 referências e 166 fontes. Nomear o
denominador mantém as afirmações honestas. Ele diz contra qual universo as ferramentas foram
checadas e, igualmente importante, o que fica fora do escopo de propósito.

## Um eixo: false-green

Toda ferramenta da família detecta uma coisa: um teste que passa verde sem proteger nada (a
[taxonomia F1-F8](taxonomy.md)). A esmagadora maioria do catálogo é uma preocupação diferente.
Misturar essas elevaria a taxa de falso-positivo, e uma ferramenta que grita lobo é desligada.

## O que fica de fora, e por quê

| Categoria | Por que fica fora | Exemplos (id do catálogo) |
|---|---|---|
| **Fragilidade / false-red** | quebra *sem* um bug real; o eixo oposto | Sensitive Equality, Brittle Assertion, Fragile Fixture, Interface/Behavior Sensitivity, Overspecified Software |
| **Higiene / manutenibilidade** | o teste ainda protege; só é difícil de ler (território de linter: ruff, ESLint, Robocop) | Assertion Roulette, Magic Number Test, Long Test, Verbose Test, Overcommented Test, Missing Assertion Message |
| **Lento / performance** | tempo de execução, invisível no arquivo | Slow Test, Slow Component Usage |
| **Design** | arquitetura, não o verde do teste | Constructor Initialization, Refused Bequest, Hard-To-Test Code |
| **Nomenclatura / docs** | legibilidade; um linter cuida | Anonymous Test, Bad Naming, Absence Of Why, Bad Comment Rate |
| **Duplicação** | manutenibilidade | Test Code Duplication, Duplicated Code, Duplicate Assert |
| **Runtime / cultura** | precisa da suíte rodando, ou é prática de time | Test Run War, Erratic Test, Manual Intervention, Frequent Debugging |

O catálogo usa o próprio esquema de id (`A16`, `S06`, `C30`), que não é o mesmo dos códigos da
família (`C1`, `C23`). Os ids acima são os do catálogo.

## A fronteira é deliberada, não acidental

Vários smells parecem fora do escopo mas têm um *proxy* de false-green que a família pega. As
ferramentas adotam a forma estaticamente provável e de baixo falso-positivo de cada fronteira e
deixam o resto:

| Smell do catálogo | O proxy que a família pega |
|---|---|
| Nondeterministic / flaky test | `C16` (tempo, aleatoriedade ou sleep descontrolados) |
| Resource Optimism | `C23` (caminho ou URL de IP hard-coded) |
| Interacting / order-dependent tests | `C24` / `C15` / `S13` (estado compartilhado) |
| Conditional Test Logic | `C1` / `C21` (uma assertion que pode nunca rodar) |
| No Assertions / Empty / Assertionless Test | `C2` / `C2b` |
| Rotten Green Test | o produto inteiro: o eixo false-green |

Alguns smells de higiene aparecem como **diagnósticos opcionais** (desligados por padrão, não
false-green): Assertion Roulette como `D1`, Magic Number como `D8`, Duplicate Assert como `D3`,
Long Test como `M2`. Onde um linter dedicado cobre um smell melhor, os scanners cedem o lugar.

## Por que isso importa para a pesquisa

Esta página é a declaração de ameaças à validade em forma pública. Precisão e recall são
reportados contra a fatia false-green do catálogo, não contra os 517 inteiros, e a tabela de
fronteira acima mostra exatamente onde a linha fica. O cruzamento completo (cada smell do
catálogo mapeado como dentro ou fora do escopo, com o motivo) vive no hub privado de pesquisa;
só o denominador e a fronteira são públicos, nunca a adjudicação crua.
