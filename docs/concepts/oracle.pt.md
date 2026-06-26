# A hierarquia do oráculo

Um oráculo é a fonte do valor esperado: aquilo contra o que o teste compara o código. Um teste
é tão honesto quanto seu oráculo. Se o valor esperado vem do próprio código, o teste confirma
que o código bate consigo mesmo, o que é verdade até quando o código está errado.

O valor esperado precisa vir de uma fonte independente do código, nesta ordem:

1. **Especificação ou requisito explícito** - um documento de spec, ticket ou RFC. O oráculo
   mais forte: precede o código e não muda quando o código muda.
2. **Contrato documentado** - uma docstring, anotações de tipo, docs de API. Mais fraco que uma
   spec mas ainda independente da implementação.
3. **Julgamento humano independente** - o testador deriva o valor esperado do requisito à mão,
   sem ler a saída atual.
4. **O código atual** - a prioridade mais baixa. É aqui que os bugs se escondem.

## Por que a ordem importa

Promover o código atual ao topo dessa hierarquia é como um bug fica congelado como "correto".
O padrão parece razoável: roda a função, captura o que ela retorna, cola isso como o valor
esperado. Daí em diante o teste passa enquanto o código continuar fazendo o que faz hoje,
incluindo o bug.

O catálogo tem vários códigos para exatamente essa inversão:

- **`C14`** - o golden file escrito a partir da saída real na primeira execução.
- **`C12`** - o teste re-implementa a fórmula de produção, então os dois lados concordam na mesma
  resposta errada.
- **`C11` / `C11a`** - o teste afirma o valor que acabou de fornecer.
- **`S7`** - a versão semântica: o valor esperado foi tirado de uma execução do código atual
  (um dict colado, uma resposta capturada).
- **Caso 18 / `S3`** - o valor esperado contradiz a spec, então o teste congelou um bug como o
  comportamento correto. Esses exigem um oráculo independente citado antes de o achado ser
  reportado.

## A regra de reporte

Para qualquer achado que dependa de o valor esperado estar errado, a skill não reporta sem
citar um oráculo. "Esse número parece errado" não basta; o achado precisa apontar para a spec,
a docstring ou a derivação que diz qual o número deveria ser. Um falso positivo aqui é caro,
então a régua é uma fonte explícita e independente.

Esse é o princípio por trás de toda a família: os scanners estáticos pegam as inversões
estruturais que um parser consegue provar (`C14`, `C12`, `C11a`), e a [passada semântica](../scanners/skill.md)
pega as que exigem ler o valor esperado contra a spec.

## O problema do oráculo

Decidir o resultado esperado é um problema de pesquisa com nome - o *problema do oráculo* (Barr, Harman,
McMinn, Shahbaz, Yoo, *The Oracle Problem in Software Testing: A Survey*, IEEE TSE 2015). Quando um
oráculo completo é difícil de construir, a literatura recorre a oráculos parciais: relações metamórficas,
verificações baseadas em propriedade ou uma referência conhecida-boa. O falsegreen adota a postura prática
acima - o valor esperado precisa ser independente do código - e roteia os casos que exigem um oráculo real
para a [skill](../scanners/skill.md), nunca reportando um achado de valor-errado sem um. Veja
[créditos](../credits.md) para as referências.
