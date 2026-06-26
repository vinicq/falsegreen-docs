# Créditos e referências

O catálogo se apoia em pesquisa publicada, não inventada. Esta página é a base do estudo: a
literatura sobre a qual o falsegreen se constrói, com DOIs onde disponíveis. Cada repositório
carrega seu próprio `CREDITS.md` com o mapa de código para fonte; o cruzamento completo do
denominador está na página [cobertura vs a literatura](concepts/denominator.md).

## Trabalho fundador: rotten green e false-green

O eixo false-green generaliza a ideia de rotten-green-test: um teste que roda em verde
enquanto sua asserção nunca executa ou nunca falha.

- Soares, E. A. *A Multimethod Study of Test Smells: Cataloging, Removal, and New Types.* PhD
  thesis, Centro de Informática, Universidade Federal de Pernambuco (UFPE), Recife, 2023. A
  fonte central: nomeia o mecanismo (Conditional Test Logic produz um resultado "passou" não
  confiável quando a asserção é pulada) e um catálogo unificado de 480 smells distintos a
  partir de 127 estudos primários.
- Delplanque, J.; Ducasse, S.; Polito, G.; Black, A. P.; Etien, A. *Rotten Green Tests.* ICSE
  2019. O trabalho que dá nome à família.
- Aranega, V. et al. *Rotten green tests in Java, Pharo and Python.* Empirical Software
  Engineering, v. 26, n. 6, p. 130, 2021.
- Martinez, M.; Etien, A.; Ducasse, S.; Fuhrman, C. *RTj: A Java framework for detecting and
  refactoring rotten green test cases.* ICSE Companion, 2020, p. 69-72.
- Kim, D. J. et al. *Studying test annotation maintenance in the wild.* ICSE 2021, p. 62-73.
- Aranda III, M.; Ribeiro, M. *Beyond Green Tests: Removing Smells From Natural Language Tests.*
  SBQS 2025, São José dos Campos.

## Catálogo e taxonomia

- van Deursen, A.; Moonen, L.; van den Bergh, A.; Kok, G. *Refactoring Test Code.* XP 2001. O
  catálogo fundador de 11 test smells mais refatorações.
- Meszaros, G. *xUnit Test Patterns: Refactoring Test Code.* Addison-Wesley, 2007. A fonte de
  Conditional Test Logic e de muitos padrões estruturais.
- Palomba, F.; Zaidman, A.; De Lucia, A. *Automatic Test Smell Detection using Information
  Retrieval Techniques.* ICSME 2018, p. 311-322. DOI [10.1109/ICSME.2018.00040](https://doi.org/10.1109/ICSME.2018.00040).
- Aljedaani, W. et al. *Test Smell Detection Tools: A Systematic Mapping Study.* EASE 2021, p.
  170-180. DOI [10.1145/3463274.3463335](https://doi.org/10.1145/3463274.3463335).

## Ferramentas de detecção e manutenibilidade

- Hauptmann, B.; Eder, S.; Junker, M.; Juergens, E.; Woinke, V. *Generating Refactoring Proposals
  to Remove Clones from Automated System Tests.* ICPC 2015, p. 115-124. DOI
  [10.1109/ICPC.2015.20](https://doi.org/10.1109/ICPC.2015.20).
- Pizzini, A.; Reinehr, S.; Malucelli, A. *Automatic Refactoring Method to Remove Eager Test
  Smell.* SBQS 2022, Curitiba. DOI [10.1145/3571473.3571478](https://doi.org/10.1145/3571473.3571478).
- Fowler, M.; Beck, K. *Refactoring: Improving the Design of Existing Code.* 2nd ed., 2019. O
  vocabulário de smells e de refatoração que preserva comportamento.

## Detecção baseada em LLM

- Peixoto, M. et al. *On the Effectiveness of LLMs for Manual Test Verifications.*
  arXiv:[2409.12405](https://arxiv.org/abs/2409.12405), 2024.
- Melo, R. et al. *Agentic LMs: Hunting Down Test Smells.*
  arXiv:[2504.07277](https://arxiv.org/abs/2504.07277), 2025.

## O denominador

- *Open Catalog of Test Smells.* UFAL / easy-software, [test-smell-catalog.readthedocs.io](https://test-smell-catalog.readthedocs.io/)
  ([repo](https://github.com/easy-software-ufal/catalog-test-smells)). 517 smells, 1621
  referências, 166 fontes. O universo com nome contra o qual precisão e revocação são medidas;
  só a fatia false-green está no escopo (veja [cobertura vs a literatura](concepts/denominator.md)).

## Linhas de base publicadas (contexto de comparação)

Resultados da literatura publicada, usados como pontos de comparação. Nossa própria avaliação
contra o denominador vive no hub de pesquisa e é liberada quando publicada.

| Fonte | Tarefa | Métrica | Valor |
|---|---|---|---|
| Soares 2023 (NLP) | detect 7 manual smells | P / R / F1 | 0.92 / 0.95 / 93.53% |
| Aranda 2025 (NLP) | detect + remove 7 manual smells | F1 | 83.70% |
| Palomba 2018 (TASTE) | detect Eager Test | F1 | 76% (structural baseline 47%) |
| Palomba 2018 (TASTE) | detect General Fixture | F1 | 67% (structural baseline 23%) |
| Melo 2025 (4 agents) | detect 5 smells (Java) | pass@5 | 96% |
| Melo 2025 (Phi-4-14B) | refactor 5 smells | pass@5 | 75.3% |
| Pizzini 2022 | remove Eager Test automatically | removal rate | 99.4% |

Um achado recorrente entre esses: detectar Conditional Test Logic é fácil (~96%), mas removê-lo
corretamente - sem trocar um false-green por um false-red - é a parte difícil (~10% em Melo
2025). Essa lacuna é exatamente o que a [hierarquia de oráculos](concepts/oracle.md) e o
[portão de correção por IA](scanners/skill.md) endereçam.

## O problema do oráculo

O problema do valor esperado tem uma referência canônica, e o órgão de padrões o trata como
cidadão de primeira classe:

- Barr, E. T.; Harman, M.; McMinn, P.; Shahbaz, M.; Yoo, S. *The Oracle Problem in Software
  Testing: A Survey.* IEEE TSE, 2015. DOI [10.1109/TSE.2014.2372785](https://doi.org/10.1109/TSE.2014.2372785).
  O syllabus ISTQB CT-AI se apoia nele.
- Segura, S. et al. *A Survey on Metamorphic Testing.* IEEE TSE, 2016; e *Metamorphic Testing:
  Testing the Untestable.* IEEE Software, 2020. Soluções para o problema do oráculo.
- Wiegers, K.; Beatty, J. *Software Requirements.* 3rd ed., 2013. Um requisito verificável é um
  para o qual você consegue construir um oráculo; um teste verde cuja especificação rastreada
  mudou é um "suspect link"; a "profecia autorrealizável" de testar contra o código é a falha
  F7 em termos de requisitos.

A [hierarquia de oráculos](concepts/oracle.md) é a resposta prática: o valor esperado precisa
vir de uma fonte independente do código.

## Vocabulário

A taxonomia se alinha ao vocabulário padrão de teste (o glossário ISTQB: test oracle, defect
vs failure, test level) e mantém o termo de produto **false-green** para o modo de falha
específico: um teste que passa enquanto não protege nada.

Uma distinção precisa, fácil de errar: em termos ISTQB um **false-positive** é um relato de
falha espúrio (um teste que falha sem um defeito real). **false-green é o oposto** - um teste
que *passa* sem proteger nada, o que habilita um **false-negative** (um defeito que escapou). A
cadeia que sustenta o produto é: um defeito não produz falha nesta execução, o teste continua
verde, o defeito vai pra produção. A porcentagem de cobertura esconde isso; o Defect Detection
Percentage (defeitos encontrados por teste sobre o total) não, e é por isso que a avaliação
reporta precisão e revocação contra a fatia false-green em vez de cobertura.

## Como citar

Se você usa o falsegreen em trabalho acadêmico, cite o repositório de produto relevante e a
literatura fundadora de rotten-green-test acima. Uma entrada de citação canônica é adicionada
aqui assim que o estudo for publicado. Veja a [fundamentação de pesquisa](research.md) para a
metodologia.
