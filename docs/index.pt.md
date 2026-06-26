# falsegreen

**Um problema, uma ferramenta: o falso positivo.** O falsegreen encontra testes que passam
em verde sem proteger nada. Um teste fica verde porque não asserta nada, asserta algo sempre
verdadeiro, nunca chega até a asserção, ou verifica a coisa errada. O código por baixo pode
estar quebrado e a suíte ainda relata sucesso.

Um teste que diz que um programa quebrado está seguro é pior que nenhum teste. Ele compra
confiança falsa, e confiança falsa entrega bug em produção. Assistentes de IA geram esses
testes em escala, e é por isso que o problema merece uma ferramenta dedicada.

Este site é a referência compartilhada para toda a família: cada código de detecção, o sinal
exato em que cada um se baseia, e um exemplo RUIM ao lado do sósia LIMPO para você ver a
diferença.

É também um projeto de pesquisa. A taxonomia é defensável, o denominador tem nome, e as
ameaças à validade estão declaradas às claras. Veja a [fundamentação de pesquisa](research.md).

[Explore o catálogo](catalog/index.md){ .md-button .md-button--primary }
[Como os códigos são classificados](concepts/taxonomy.md){ .md-button }
[Fundamentação de pesquisa](research.md){ .md-button }

## A família

Quatro ferramentas, um catálogo. Os três scanners estáticos provam o que um parser consegue
ver, cada um na sua linguagem. A skill é a passagem semântica que lê a intenção, e um superset
das três.

| Ferramenta | Linguagem | Técnica |
|---|---|---|
| [falsegreen](scanners/python.md) | Python / pytest | varredura de AST, zero dependências |
| [falsegreen-js](scanners/javascript.md) | JavaScript / TypeScript | API do compilador TypeScript |
| [robotframework-falsegreen](scanners/robot.md) | Robot Framework | modelo `robot.api` |
| [falsegreen-skill](scanners/skill.md) | Python, TS, JS, Robot | passagem semântica via LLM (J1-J6) |

Os códigos compartilham um id quando o conceito coincide entre linguagens: `C5` é a asserção
sempre verdadeira em Python, JavaScript e Robot por igual. Padrões específicos de cada
linguagem ganham sua própria série (`JS*` para JavaScript, `R*` para Robot, `S*` para a
passagem semântica).

## Três camadas de false-green

Nem todo teste false-green é visível a um parser. O catálogo se organiza em torno de onde o
problema mora, porque é isso que decide qual ferramenta consegue pegá-lo.

1. **Estático, por arquivo.** A asserção é vazia, sempre verdadeira, inalcançável, ou
   engolida. Um parser prova isso sem rodar nada. É o que os três scanners fazem.
2. **Projeto e CI.** O arquivo está limpo mas a suíte ainda mente: uma config que deixa uma
   execução vazia passar, um portão de cobertura que nunca é exigido, warnings que nunca viram
   erros. É o modo `--config-audit`.
3. **Semântico e em tempo de execução.** O oráculo é circular, o valor esperado contradiz a
   especificação, o teste faz mock da própria unidade que diz testar. Nenhuma AST enxerga
   intenção. A skill lê isso; um portão ao vivo confirma com teste de mutação.

Veja [escopo e honestidade](concepts/scope.md) para o que cada camada pode e não pode provar.

## Como ler um código

Toda entrada do catálogo carrega os mesmos campos:

- **O que detecta** - a falha em termos simples.
- **Sinal** - o gatilho sintático concreto em que a ferramenta se baseia, e quando ela
  deliberadamente se segura para evitar um falso positivo.
- **RUIM / LIMPO** - o padrão sinalizado ao lado de um sósia que está correto, para que a
  fronteira fique visível.
- **Julgamento** (J1-J6) e **família** (F1-F8) - onde o código se encaixa na
  [taxonomia](concepts/taxonomy.md).
- **Confiança** - HIGH bloqueia, LOW avisa, OFF é só diagnóstico.

A regra que guia toda a família: **um falso positivo é pior que um escape.** Uma ferramenta
que grita lobo é desligada, e uma ferramenta desligada não protege nada. Cada código é
ajustado para disparar num false-green real e ficar quieto no sósia que se parece com ele.
