# Catálogo JavaScript / TypeScript

Os códigos implementados por [falsegreen-js](../scanners/javascript.md): uma varredura estática via a
API do compilador TypeScript, agnóstica de runner entre Jest, Vitest, Mocha+Chai, Jasmine, AVA,
node:test, Cypress, Playwright e Testing Library. Cobre `.js`, `.jsx`, `.ts`, `.tsx`,
`.mjs`, `.cjs`, `.mts`, `.cts`.

Os códigos compartilham um id com [Python](python.md) onde o conceito coincide; códigos `JS*` são
específicos do ecossistema. Confiança: **ALTO** bloqueia, **BAIXO** avisa. Julgamentos são
[J1-J6](../concepts/judgments.md).

Cada código emitido tem sua própria entrada abaixo. O índice leva direto a cada uma.

## Índice

| Código | Conf | J | Resumo |
|---|---|---|---|
| [C2](#c2) | ALTO | J1 | corpo de teste vazio |
| [C2b](#c2b) | BAIXO | J1 | chama a unidade mas nunca afirma |
| [C5](#c5) | ALTO | J2 | sempre verdadeira (`expect(true).toBe(true)`) |
| [C6](#c6) | BAIXO | J4 | verificação fraca (`toBeTruthy`, `.length > 0`) |
| [C7](#c7) | ALTO | J2 | autocomparação (`expect(x).toBe(x)`) |
| [C8](#c8) | BAIXO | J4 | igualdade exata em um float |
| [C8b](#c8b) | BAIXO | J4 | `toBeCloseTo` sem argumento de precisão |
| [C9](#c9) | BAIXO | J4 | `toThrow()` sem tipo de erro ou mensagem |
| [C11a](#c11a) | BAIXO | J2 | literal autoconfirmante ligado da chamada sob teste |
| [C16](#c16) | BAIXO | J1 | depende de tempo, aleatoriedade ou um timer fixo |
| [C18](#c18) | BAIXO | J2 | igualdade de string (`String(x)`/`JSON.stringify`) |
| [C20](#c20) | ALTO | J1 | asserção morta depois de `return`/`throw` |
| [C21](#c21) | BAIXO | J1 | nenhuma asserção roda incondicionalmente |
| [C23](#c23) | BAIXO | J6 | lê um arquivo real num caminho literal / URL fixa |
| [C37](#c37) | BAIXO | J4 | caso duplicado de `it.each`/`test.each` |
| [C44](#c44) | ALTO | J2 | tautologia numérica sobre um comprimento (`.length >= 0`) |
| [C48](#c48) | BAIXO | J1 | dark patch: vira um flag de modo de teste e afirma |
| [CC](#cc) | BAIXO | J1 | asserção comentada |
| [JS1](#js1) | ALTO | J1 | teste focado pula o resto da suíte |
| [JS2](#js2) | ALTO | J1 | `expect` sem matcher |
| [JS3](#js3) | BAIXO | J2 | o snapshot é a única asserção |
| [JS4](#js4) | BAIXO | J1 | teste pulado (`it.skip`/`xit`/`it.todo`) |
| [JS5](#js5) | BAIXO | J1 | query/evento async sem await |
| [JS6](#js6) | ALTO | J1 | `describe`/suíte vazia |
| [JS7](#js7) | BAIXO | J1 | asserção em `setTimeout`/`then` sem await |
| [JS8](#js8) | BAIXO | J3 | mocka a unidade sob teste e a afirma direto |
| [JS9](#js9) | ALTO | J1 | asserção em um ramo literal morto |
| [JS11](#js11) | BAIXO | J1 | try/catch engole a asserção |
| [JS13](#js13) | BAIXO | J4 | query usada como instrução solta, nunca afirmada |
| [JS15](#js15) | BAIXO | J4 | comparação embrulhada em um booleano |
| [JS17](#js17) | BAIXO | J1 | bloco de teste comentado |
| [JS18](#js18) | BAIXO | J1 | callback `done` em vez de async/await |
| [JS21](#js21) | ALTO | J1 | matcher referenciado mas nunca chamado |
| [JS22](#js22) | ALTO | J1 | tabela `it.each`/`test.each` vazia |
| [JS23](#js23) | ALTO | J1 | `expect.assertions(N)` que o corpo não satisfaz |
| [JS24](#js24) | BAIXO | J4 | query do Cypress sem asserção |
| [JS25](#js25) | ALTO | J1 | a única asserção está dentro de um callback de iterador |
| [JS26](#js26) | BAIXO | J1 | fake timers instalados mas nunca avançados |
| [JS27](#js27) | BAIXO | J3 | `toHaveBeenCalled*` é o único oráculo num dublê local |
| [JS29](#js29) | BAIXO | J6 | cadeia `resolves`/`rejects` é uma instrução solta |
| [JS30](#js30) | ALTO | J2 | asserção literal contra literal |
| [JS31](#js31) | BAIXO | J1 | try/catch engole um throw do SUT, sem asserção no erro |
| [D1](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J4 | roleta de asserções |
| [D3](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J4 | assert duplicado |
| [D4](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J4 | casos de `it.each` sem título |
| [D6](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J4 | `console.*` no corpo do teste |
| [D7](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J4 | teste anônimo |
| [D8](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J4 | número mágico em uma asserção |
| [M2](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J5 | corpo de teste longo demais |
| [PL7](#pl7) | BAIXO | J5 | sem gate de cobertura |
| [PL8](#pl8) | BAIXO | J5 | `bail` interrompe a execução cedo |
| [PL10](#pl10) | BAIXO | J1 | `passWithNoTests` deixa uma suíte vazia passar |

Um código mantém seu id entre linguagens onde o smell é o mesmo; o sinal abaixo é a forma
JavaScript. Dois ids compartilhados diferem em sentido por linguagem, veja a [página
Python](python.md): **C31** não existe aqui; **C44** é a tautologia numérica de comprimento aqui e
no Python, ampliada no [Robot](robot.md#c44) para qualquer asserção de biblioteca vazia.

## Códigos compartilhados (mesmo conceito do Python)

Estes espelham as entradas do [catálogo Python](python.md); o sinal é a forma JavaScript.

### C2 - corpo de teste vazio { #c2 }
`J1` · ALTO · F1

Um teste sem matcher, sem `expect`, sem nenhuma asserção. Sempre verde.

### C2b - chama a unidade mas nunca afirma { #c2b }
`J1` · BAIXO · F1

O teste chama código de produção mas nenhum matcher segue. A verificação simplesmente falta.

### C5 - asserção sempre verdadeira { #c5 }
`J2` · ALTO · F3

`expect(true).toBe(true)`, `assert(1)`: estruturalmente garantida a passar, não protege nada.

### C6 - verificação fraca { #c6 }
`J4` · BAIXO · F4

`toBeTruthy()` / `toBeDefined()` ou um `.length > 0`: só que algo voltou, não o valor.

### C7 - autocomparação { #c7 }
`J2` · ALTO · F3

`expect(x).toBe(x)`: os dois lados são a mesma expressão, sempre iguais.

### C8 - igualdade exata em um float { #c8 }
`J4` · BAIXO · F4

`expect(compute()).toBe(3.14159)`: a aritmética de ponto flutuante torna a igualdade exata pouco
confiável. Use `toBeCloseTo`.

### C8b - `toBeCloseTo` sem argumento de precisão { #c8b }
`J4` · BAIXO · F4

`toBeCloseTo(x)` usa por padrão tolerância de 2 dígitos, que pode ser frouxa demais para o valor sob
teste. Passe uma precisão explícita.

### C9 - `toThrow()` sem tipo de erro ou mensagem { #c9 }
`J4` · BAIXO · F4

`expect(fn).toThrow()` sem argumento aceita qualquer erro, inclusive um de um typo dentro do teste.

### C11a - literal autoconfirmante { #c11a }
`J2` · BAIXO · F3

O valor esperado é ligado da mesma chamada sob teste: `const e = foo(); expect(foo()).toBe(e)`. O
oráculo confirma a si mesmo. Ligue o valor esperado de forma independente.

### C16 - depende de tempo, aleatoriedade ou um timer fixo { #c16 }
`J1` · BAIXO · F6

`Date.now` / `new Date()` / `Math.random` / `crypto.randomUUID` / `getRandomValues`, ou um timer
fixo. Congele o tempo e semeie a aleatoriedade.

### C18 - igualdade de string { #c18 }
`J2` · BAIXO · F4

Compara `String(x)` / `JSON.stringify(x)` / um template literal com um literal de string. O formato
é detalhe de implementação; afirme o valor.

### C20 - asserção morta depois de um terminador { #c20 }
`J1` · ALTO · F2

Uma asserção depois de `return` / `throw` / `process.exit` / um `switch` exaustivo. A alcançabilidade
estruturada no nível de bloco (cfg.ts) prova que nunca roda.

### C21 - nenhuma asserção roda incondicionalmente { #c21 }
`J1` · BAIXO · F2

Toda asserção está dentro de um ramo; nenhuma fica na espinha garantida do teste. A mesma
alcançabilidade do cfg.ts decide isso, e uma asserção morta não mascara o caso.

### C23 - lê um arquivo real num caminho literal ou URL fixa { #c23 }
`J6` · BAIXO · F6

`readFileSync("/etc/...")` ou uma URL fixa no código: mystery guest. Use uma fixture ou arquivo
temporário.

### C37 - caso duplicado de `it.each` / `test.each` { #c37 }
`J4` · BAIXO · F8

A mesma linha de argumentos aparece duas vezes na tabela; a duplicata roda o mesmo cenário e não
adiciona cobertura.

### C44 - tautologia numérica sobre um comprimento { #c44 }
`J2` · ALTO · F3

`expect(x.length).toBeGreaterThanOrEqual(0)`: um comprimento nunca é negativo, então a comparação é
sempre verdadeira.

### C48 - dark patch: vira um flag de modo de teste e afirma { #c48 }
`J1` · BAIXO · F2

O teste vira um flag de modo de teste (`process.env.NODE_ENV = "test"`, `process.env.TESTING = "1"`,
`settings.TESTING = true`) e depois afirma, então exercita o ramo só-de-teste do produto em vez do
comportamento real. Paridade com o `C48` do Python.

### CC - asserção comentada { #cc }
`J1` · BAIXO · F2

Uma linha no corpo é um `expect(...)` comentado: a verificação foi desligada e deixada.

## Códigos específicos de JavaScript

### JS1 - teste focado pula o resto da suíte { #js1 }
`J1` · ALTO · F5

`it.only` / `fit` / `describe.only` exclui em silêncio todos os outros testes do arquivo da execução.

=== "RUIM"
    ```javascript
    it.only('adds', () => { expect(add(2, 3)).toBe(5); });
    it('subtracts', () => { /* never runs while .only is present */ });
    ```
=== "LIMPO"
    ```javascript
    it('adds', () => { expect(add(2, 3)).toBe(5); });
    it('subtracts', () => { expect(sub(3, 2)).toBe(1); });
    ```

### JS2 - expect sem matcher { #js2 }
`J1` · ALTO · F1

`expect(value)` sem uma chamada de matcher. Nada é afirmado; a linha é um no-op.

=== "RUIM"
    ```javascript
    test('result', () => { expect(getResult()); });   // JS2 - sem matcher
    ```
=== "LIMPO"
    ```javascript
    test('result', () => { expect(getResult()).toBe(42); });
    ```

### JS3 - o snapshot é a única asserção { #js3 }
`J4` · BAIXO · F3

Um teste cuja única verificação é `toMatchSnapshot()` / `toMatchInlineSnapshot()`. A baseline é
gerada a partir da saída, então detecta mudança, não correção.

### JS4 - teste pulado { #js4 }
`J1` · BAIXO · F5

`it.skip` / `xit` / `it.todo` deixado no lugar. Excluído em silêncio da execução.

### JS5 - query ou evento async não aguardado { #js5 }
`J1` · BAIXO · F2

Uma chamada do tipo `promise` ou `value-only` (`findBy*`, `waitFor`, `userEvent`, um `expect().resolves`
solto) é descartada como instrução nua, então a asserção seguinte lê um momento defasado.
A detecção passa pelo registro de oráculos.

=== "RUIM"
    ```javascript
    it('shows the row', () => {
      screen.findByText('Ada');                 // promise dropped
      expect(screen.getByRole('row')).toBeInTheDocument();
    });
    ```
=== "LIMPO"
    ```javascript
    it('shows the row', async () => {
      await screen.findByText('Ada');
      expect(screen.getByRole('row')).toBeInTheDocument();
    });
    ```

### JS6 - describe / suite vazio { #js6 }
`J1` · ALTO · F1

Um bloco `describe`/`suite` sem teste dentro. Não contribui com nada e pode ser lido como cobertura.

### JS7 - asserção em um setTimeout / callback then não aguardado { #js7 }
`J1` · BAIXO · F2

A asserção vive em um callback de `setTimeout`/`setInterval` (braço de timer) ou em um
`.then`/`.catch`/`.finally` solto (braço de promise), então pode rodar depois de o teste reportar verde.

=== "RUIM"
    ```javascript
    it('fires later', () => {
      setTimeout(() => { expect(handler).toHaveBeenCalled(); }, 100);  // runs after the test ends
    });
    ```
=== "LIMPO"
    ```javascript
    it('fires later', async () => {
      jest.useFakeTimers();
      schedule();
      jest.runAllTimers();
      expect(handler).toHaveBeenCalled();
    });
    ```

### JS8 - faz mock da unidade sob teste e a afirma diretamente { #js8 }
`J3` · BAIXO · F4

O teste faz mock da própria função que afirma testar, depois afirma o valor do mock. Ele testa a
configuração do mock, não o código. (A forma semântica, mais profunda, é o caso 10 / `S12`.)

### JS9 - asserção em um ramo literal morto { #js9 }
`J1` · ALTO · F2

`if (false) { expect(...) }` ou o `else` de `if (true)`. A asserção é inalcançável.

### JS11 - try/catch engole a asserção { #js11 }
`J1` · BAIXO · F2

A asserção fica em um `try` cujo `catch` absorve o erro lançado (loga ou ignora), então uma
falha nunca se propaga.

=== "RUIM"
    ```javascript
    it('throws', () => {
      try { callUnit(); expect(true).toBe(false); } catch (e) { console.log(e); }
    });
    ```
=== "LIMPO"
    ```javascript
    it('throws', () => { expect(() => callUnit()).toThrow(RangeError); });
    ```

### JS13 - query usada como instrução solta, nunca afirmada { #js13 }
`J4` · BAIXO · F1

`getBy*` / `queryBy*` chamado como instrução nua. A query roda mas nada é verificado nela.

### JS15 - comparação envolvida em um booleano { #js15 }
`J4` · BAIXO · F4

`expect(a === b).toBe(true)`. O booleano colapsa o diff: na falha a mensagem é
`false !== true`, sem valores. Afirme os valores diretamente.

=== "RUIM"
    ```javascript
    expect(user.id === 1).toBe(true);   // JS15
    ```
=== "LIMPO"
    ```javascript
    expect(user.id).toBe(1);
    ```

### JS17 - bloco de teste comentado { #js17 }
`J1` · BAIXO · F2

`// it('...', ...)` deixado no arquivo. O teste não roda.

### JS18 - callback done em vez de async/await { #js18 }
`J1` · BAIXO · F2

Um teste com callback `done` onde `done()` precede ou fica no mesmo nível da asserção, então o
runner conclui antes de a asserção lançar.

### JS21 - matcher referenciado mas nunca chamado { #js21 }
`J1` · ALTO · F2

`expect(x).toBe` sem `()`. O matcher é um acesso a propriedade; a verificação nunca roda. Irmão de
JS2.

=== "RUIM"
    ```javascript
    expect(result).toBe;          // JS21 - faltando (), não faz nada
    ```
=== "LIMPO"
    ```javascript
    expect(result).toBe(42);
    ```

### JS22 - tabela it.each / test.each vazia { #js22 }
`J1` · ALTO · F5

`it.each([])(...)`. Zero casos são gerados, então o teste nunca roda. Paralelo de `C45`.

### JS23 - expect.assertions(N) que o corpo não consegue satisfazer { #js23 }
`J1` · ALTO · F5

`expect.assertions(N)` com um `N` numérico maior que as chamadas `expect()` incondicionais,
alcançáveis e não-aninhadas que podem rodar. A garantia que o autor escreveu para se proteger de
uma asserção async perdida em silêncio já nasce morta. Dispara só num déficit provável de `N`
literal: um `expect` em loop, em ramo, num `.then`/callback, ou num helper torna a contagem
indeterminada e suprime o achado. `expect.hasAssertions()` não tem contagem e é ignorado.

### JS24 - query do Cypress sem asserção { #js24 }
`J4` · BAIXO · F4

Uma cadeia de query do Cypress (`cy.get`/`cy.find`/`cy.contains`) usada como statement sem
`.should`/`.and` no fim e sem `expect` dentro de um callback `.then`. A query produz um subject que
nunca é asserido, o análogo cy.* da JS13. Comandos de ação (`click`/`type`/`visit`/...) fazem
trabalho em vez de consultar, então uma cadeia terminada num deles fica limpa, assim como uma
terminada em `.should`/`.and`.

### JS25 - a única asserção fica dentro de um callback de iterador de array { #js25 }
`J1` · ALTO · F2

A única `expect` fica dentro de um callback de `forEach` / `map` / `filter` / `some` / `every` /
`flatMap`. Numa coleção vazia o callback nunca roda, então o teste passa sem ter afirmado nada.
Afirme o comprimento antes, ou tire ao menos uma verificação para fora do iterador.

=== "RUIM"
    ```javascript
    it('all valid', () => {
      rows.forEach(r => expect(r.valid).toBe(true));   // JS25 - nada roda se rows for []
    });
    ```
=== "LIMPO"
    ```javascript
    it('all valid', () => {
      expect(rows.length).toBeGreaterThan(0);
      rows.forEach(r => expect(r.valid).toBe(true));
    });
    ```

### JS26 - fake timers instalados mas nunca avançados { #js26 }
`J1` · BAIXO · F2

`jest.useFakeTimers()` / `vi.useFakeTimers()` (ou fake timers do `sinon`) é configurado, mas o
relógio nunca é avançado (`runAllTimers`, `advanceTimersByTime`, `tick`). O callback agendado nunca
dispara, então a asserção lê estado não-modificado e passa pelo motivo errado.

### JS27 - toHaveBeenCalled* é o único oráculo sobre um dublê criado localmente { #js27 }
`J3` · BAIXO · F4

A única asserção é `toHaveBeenCalled` / `toHaveBeenCalledWith` / `toHaveBeenCalledTimes` sobre um
mock criado no teste (`jest.fn()` / `vi.fn()`). Ela verifica a fiação do próprio teste, que o
dublê foi chamado, não que a unidade produziu o resultado certo.

### JS29 - cadeia resolves / rejects como instrução solta { #js29 }
`J1` · BAIXO · F2

`expect(p).resolves.toBe(...)` / `.rejects...` escrito como uma instrução que não é nem `await`ada
nem `return`ada. O matcher devolve uma promise; o teste termina verde antes de ela resolver, então
uma rejeição posterior se perde.

=== "RUIM"
    ```javascript
    it('resolves', () => {
      expect(load()).resolves.toBe(42);    // JS29 - não aguardado nem retornado
    });
    ```
=== "LIMPO"
    ```javascript
    it('resolves', async () => {
      await expect(load()).resolves.toBe(42);
    });
    ```

### JS30 - asserção literal contra literal { #js30 }
`J2` · ALTO · F3

Ambos os operandos são fixos em tempo de parse: `expect(2).toBe(3)`, chai `expect(1).to.equal(1)`.
A asserção não toca a unidade sob teste, então é sempre-verdadeira ou sempre-falsa por construção,
nunca uma verificação do comportamento real.

### JS31 - try/catch engole um possível throw sem asserção sobre a exceção { #js31 }
`J1` · BAIXO · F2

Um `try` chama a unidade e o `catch` não relança nem afirma nada sobre o erro. Uma unidade que
para de lançar ainda passa verde. Irmã da JS11, onde o que é engolido é um `expect`; aqui não há
asserção nenhuma no caminho capturado.

=== "RUIM"
    ```javascript
    it('throws on bad input', () => {
      try { parse('bad'); } catch (e) { /* JS31 - engolido, nada afirmado */ }
    });
    ```
=== "LIMPO"
    ```javascript
    it('throws on bad input', () => {
      expect(() => parse('bad')).toThrow(SyntaxError);
    });
    ```

## Armadilhas de alto valor com evidência

O scanner também pega um conjunto de false-greens específicos de idioma documentados nos estudos
empíricos de JS/TS:

- **Armadilha de caixa em header HTTP** (`J4`): o supertest deixa em minúsculas os headers de resposta, então
  `res.headers['Content-Type']` é sempre `undefined`. Use a chave em minúsculas.
- **Coerção por NOT bit a bit** (`J4`): `~~res.header['content-length']` transforma um header ausente em
  `0`, passando em silêncio quando o valor esperado é `0`.
- **Oráculo de campo autorreferente** (`J2`): `expect(result).toEqual([{ createdAt: result[0].createdAt }])`
  compara um campo consigo mesmo. Use um valor conhecido fixo.
- **Tautologia assina-depois-verifica** (`J2`): assinar um JWT e verificá-lo com a mesma chave passa
  mesmo que `verify()` pule a checagem, a menos que pareado com um teste negativo de chave errada.

## Camada de projeto - auditoria de config (PL)

Emitidos por `--config-audit`, não pela varredura por arquivo. A suíte fica verde por configuração
do runner, não por um smell dentro de algum arquivo de teste.

### PL7 - sem gate de cobertura { #pl7 }
`J5` · BAIXO · F8

Nenhum `coverageThreshold` / `coverage.thresholds` está configurado, então a cobertura pode cair a
zero e a suíte ainda passa. Adicione um limiar de cobertura.

### PL8 - `bail` interrompe a execução cedo { #pl8 }
`J5` · BAIXO · F5

`bail` está ligado, então a execução para nas primeiras falhas e a contagem de testes reportada fica
incompleta.

### PL10 - `passWithNoTests` deixa uma suíte vazia passar { #pl10 }
`J1` · BAIXO · F5

`passWithNoTests` deixa uma suíte vazia ou totalmente filtrada reportar verde, então um glob mal
configurado ou um arquivo apagado passa despercebido.

## Códigos de diagnóstico (opcionais, OFF por padrão)

Família F8: não é false-green (o teste ainda protege), mostrado só numa passagem de diagnóstico.

| Código | O que sinaliza |
|---|---|
| **D1** | roleta de asserções: muitas asserções num teste, nenhuma identificando qual falhou |
| **D3** | assert duplicado: a mesma asserção aparece mais de uma vez |
| **D4** | casos de `it.each` / `test.each` sem título: um caso que falha é nomeado só pelo índice |
| **D6** | `console.*` no corpo do teste: artefato de debug que contorna o oráculo |
| **D7** | teste anônimo: descrição vazia ou ausente |
| **D8** | número mágico em uma asserção: um literal numérico nu em vez de uma constante nomeada |
| **M2** | corpo de teste longo demais: passa do limiar de linhas |

## Parecidos: NÃO sinalizar

- `expect(fn).not.toThrow()` depois de uma chamada de setup: a ausência de exceção é a asserção
  significativa.
- `expect(emitter).toHaveProperty('on')`: checagem de duck-typing; a presença da interface é o contrato.
- `expectTypeOf(v).toEqualTypeOf<T>()`: uma asserção de tipo em tempo de compilação; falha no `tsc`, não C5.
- `expect(result).toMatchObject({ kind: 'error' })` sobre uma união discriminada: o discriminante
  determina o ramo; não C6.
- `vi.mocked(fn)` / `jest.mocked(fn)`: wrappers de mock tipados, o equivalente TS de `autospec`; não
  C13b.
- `done()` depois de uma série de asserções reais: o smell é `done()` *antes* da asserção, não
  depois.
