# Visão geral do catálogo

Todo código de detecção da família, num só lugar. Cada página de linguagem abre com uma tabela
de índice compacta (código, confiança, julgamento, resumo) que leva direto à entrada daquele
código abaixo. As entradas correm um código de cada vez, agrupadas por família, cada uma dizendo
o sinal exato que dispara a regra com um exemplo RUIM ao lado do parecido LIMPO, para deixar a
fronteira visível.

Use a caixa de busca (canto superior direito) para pular para qualquer código por id (`C5`, `JS21`, `R2`) ou por palavra-chave
(`mock`, `skip`, `always-true`). Todo código tem sua própria âncora, então relatórios e ferramentas podem apontar direto
para ele.

## As páginas

- [Python](python.md) - os códigos estruturais `C*`, a camada de auditoria de config `PL*` e o
  grupo de diagnóstico, como implementados pelo scanner [falsegreen](../scanners/python.md).
- [JavaScript / TypeScript](javascript-typescript.md) - os códigos compartilhados `C*`, os `JS*`
  específicos do ecossistema, a camada `PL*` e o grupo de diagnóstico, de
  [falsegreen-js](../scanners/javascript.md).
- [Robot Framework](robot.md) - os códigos compartilhados `C*`, os `R*`, a camada `PL*` e o grupo
  de diagnóstico, de [robotframework-falsegreen](../scanners/robot.md).
- [Semântico (LLM)](semantic.md) - os casos canônicos mais a série `S1-S21` que só a
  [skill](../scanners/skill.md) consegue pegar.
- [Gherkin e Tavern](gherkin-tavern.md) - a passagem semântica, baseada em texto, sobre arquivos `.feature` e
  `*.tavern.yaml`.

Três agrupamentos se repetem nas páginas estruturais: os códigos por arquivo (as séries `C*`,
`JS*`, `R*`), a **camada de projeto** (`PL*`), emitida pela passagem de auditoria de config quando
a suíte fica verde por configuração do runner e não por um smell dentro de algum arquivo, e o
**grupo de diagnóstico** (`D*`, `M2`), opcional e OFF por padrão, que sinaliza higiene que não é
false-green.

## Como um código é compartilhado entre linguagens

Um código mantém seu id onde o conceito é o mesmo. `C5` é a asserção sempre verdadeira, seja
escrita `assert True` em Python, `expect(true).toBe(true)` em JavaScript, ou
`Should Be True    ${TRUE}` em Robot. O sinal muda por linguagem; o modo de falha
não. Padrões específicos de linguagem ganham sua própria série: `JS*` para idiomas JavaScript, `R*` para
construções Robot, `S*` para a passagem semântica.

## A legenda

Cada entrada é marcada com três eixos.

**Confiança** decide o que a ferramenta faz com o achado:

| Confiança | Significado | Efeito |
|---|---|---|
| **ALTO** | um false-green definitivo | bloqueia (saída não-zero) |
| **BAIXO** | um smell provável, operador confirma | avisa |
| **OFF** | só diagnóstico | mostrado sob demanda, nunca bloqueia |

**Julgamento** (J1-J6) nomeia a garantia quebrada. Veja [julgamentos](../concepts/judgments.md).

| | |
|---|---|
| **J1** | a asserção não roda |
| **J2** | o valor esperado não vem de um oráculo independente |
| **J3** | a unidade real não é exercitada |
| **J4** | a asserção é insuficiente |
| **J5** | o teste se acopla a detalhes internos |
| **J6** | o teste não passa isolado |

**Família** (F1-F8) nomeia o modo de falha. Veja [taxonomia](../concepts/taxonomy.md).

| | |
|---|---|
| **F1** | não verifica nada | **F5** | sai da contagem |
| **F2** | a verificação nunca roda | **F6** | passa ou falha por sorte |
| **F3** | a verificação é sempre verdadeira | **F7** | oráculo circular ou semântico |
| **F4** | verifica a coisa errada | **F8** | higiene, não false-green |

## Uma nota sobre os parecidos

Para a maioria dos códigos o catálogo também mostra o que *não* sinalizar: o padrão que lembra o smell
mas está correto. Uma autocomparação que também testa `__eq__`, uma checagem de truthiness na camada
HTTP, um wrapper de mock tipado. Essas exceções são o motivo de a taxa de falso-positivo se manter baixa, e
estão listadas em cada página.
