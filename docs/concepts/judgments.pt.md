# Julgamentos (J1-J6)

Um julgamento é uma pergunta feita a um único teste. Seis perguntas decidem se um teste
realmente protege alguma coisa. Cada código do catálogo carrega o julgamento que responde, então
um achado nunca é só "algo parece estranho": ele nomeia qual garantia o teste deixa de oferecer.

| Julgamento | A pergunta | Um teste falha quando |
|---|---|---|
| **J1** | A assertion roda? | a verificação está ausente, inalcançável, engolida ou pulada |
| **J2** | O valor esperado vem de um oráculo independente? | o valor esperado é sempre-verdadeiro, autorreferente ou copiado da saída |
| **J3** | A unidade real é exercitada? | o teste afirma um mock, um stub ou o próprio setup |
| **J4** | A assertion é suficiente? | a verificação é fraca ou ampla demais para falhar no defeito real |
| **J5** | Está livre de acoplamento a internos? | o teste lê campos privados ou detalhe de implementação |
| **J6** | Passa em isolamento? | o resultado depende de ordem, estado compartilhado, tempo ou aleatoriedade |

## Como um julgamento vira um código

O julgamento é o *porquê*; o código é o *que uma ferramenta consegue provar*. Um julgamento
cobre vários códigos entre linguagens:

- **J1** (a assertion não roda) cobre o teste vazio (`C2`), a assertion depois de um
  `return` (`C20`), o `try/except` que engole (`C3`), a verificação comentada (`CC`), o
  teste pulado (`C32`) e os equivalentes em JS e Robot.
- **J2** (o valor esperado não é independente) cobre a assertion sempre-verdadeira (`C5`), a
  auto-comparação (`C7`), a tautologia numérica (`C44`) e o golden file copiado da saída
  (`C14`).
- **J3** (a unidade real não é exercitada) cobre fazer mock da unidade sob teste (caso 10), o
  literal autoconfirmatório (`C11a`) e o patch da lógica central (`S12`).
- **J4** (a assertion é insuficiente) cobre a verificação fraca de truthiness (`C6`), o
  `pytest.raises(Exception)` amplo (`C9`) e o oráculo grosseiro que a skill aponta (`S4`).
- **J5** (acoplamento a internos) cobre ler campos privados com prefixo underscore e
  comparações de string/repr que se prendem a detalhe de implementação.
- **J6** (não passa em isolamento) cobre estado mutável compartilhado (`C24`), dependência de
  ordem (caso 15), tempo ou aleatoriedade descontrolados (`C16`) e decorators de flaky-retry
  (`C35`).

## Por que seis e não um

Um único veredito "esse teste é bom?" esconde o motivo e convida discussão. Dividir em seis
perguntas independentes torna cada achado defensável: a ferramenta aponta para exatamente uma
garantia quebrada, mostra o sinal e deixa as outras cinco de fora. Também mantém os falsos
positivos baixos, porque um padrão só dispara quando falha claramente um julgamento específico,
não num senso vago de smell.

A passada semântica se apoia mais em J2, J3 e J4, porque essas exigem ler o valor esperado
contra a especificação e o código de produção, o que nenhum parser faz. Os scanners estáticos
ficam com os casos J1, J5 e J6 que um parser consegue provar, mais a fatia estrutural de J2 e J4.
