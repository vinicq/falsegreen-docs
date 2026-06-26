# Escopo e honestidade

A família é construída sobre uma definição: um teste false-green reporta sucesso enquanto o
código que ele cobre pode estar quebrado. Tudo dentro dessa linha está no escopo. Tudo fora
dela é nomeado aqui, com o motivo de ficar de fora, porque uma ferramenta que se estende demais
perde a confiança que a torna útil.

## O que os scanners estáticos provam

Um parser prova o que é estruturalmente verdadeiro num único arquivo, sem rodar nada:

- a assertion está ausente, inalcançável ou engolida (F1, F2);
- a assertion é sempre verdadeira por construção (F3);
- o teste some da execução (F5, a fatia por arquivo);
- um proxy estático de não-determinismo: um caminho hard-coded, um relógio não congelado (F6, parcial).

Isso é rápido, determinístico e não tem falsos negativos dentro das próprias regras. Também tem
um teto: cerca de 90% dos *mecanismos* de false-green que uma sintaxe consegue expressar.
Forçar mais códigos aqui troca sinal por ruído.

## O que a skill acrescenta

A [passada semântica](../scanners/skill.md) lê o teste contra a intenção dele, a spec e o
código de produção. Ela pega o que nenhum parser vê (F4, F7):

- o valor esperado contradiz a spec;
- o teste faz mock da unidade que diz testar;
- o oráculo é grosseiro demais para falhar no defeito real;
- a assertion verifica uma propriedade irrelevante.

A skill é o superconjunto: ela carrega todo código estrutural mais os semânticos. Um falso
positivo aqui ainda é pior que uma falha, então os achados semânticos mostram o raciocínio e
citam um oráculo antes de reportar.

## O que exige runtime, e não é prometido estaticamente

Alguns modos de false-green só aparecem quando a suíte roda:

- `python -O` removendo `assert`, um erro de coleta reportado como "0 tests passed", um passo de
  CI que roda um subconjunto e reporta verde. Essas são a fatia de runtime de F5.
- Se um teste reforçado de fato falha num bug específico. Esse é o
  [gate de AI-fix](../scanners/skill.md), provado com mutation testing (mutmut, cosmic-ray,
  Stryker), que a skill nunca roda por conta própria.

Esses são documentados, não afirmados. Uma ferramenta que os prometesse estaticamente estaria
mentindo.

## O que está deliberadamente fora

- **False-red, fragilidade, flakiness.** Testes que quebram *sem* um bug real são o eixo
  oposto. Misturá-los com false-green produz ruído e contradiz a definição do produto.
  `C8` (igualdade exata de float) e `C16` (fontes de não-determinismo) são os dois proxies
  estáticos que ficam do lado false-green da linha; o resto fica de fora.
- **Higiene pura.** Código morto, argumentos não usados, métodos longos, docs faltando. Esses
  são F8 e pertencem a linters dedicados (ruff, ESLint, Robocop). O grupo de diagnóstico mostra
  alguns sob demanda, desligado por padrão.
- **Cobertura, performance, cultura.** Testes lentos, test-run war, gates humanos de qualidade.
  Invisíveis no arquivo; uma questão de processo ou runtime, não um padrão de false-green.

O resumo honesto em cada README de scanner diz a mesma coisa: a estática prova a fatia
estrutural, a skill lê a fatia semântica, runtime fica fora de banda. Cada ferramenta declara o
que não faz, para que o verde que ela te dá signifique algo.

Para o quadro completo do que fica de fora e por quê, medido contra a literatura publicada de
test-smells, veja [cobertura versus a literatura](denominator.md).
