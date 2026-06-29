# Catálogo JavaScript / TypeScript

Os códigos implementados por [falsegreen-js](../scanners/javascript.md): uma varredura estática via a
API do compilador TypeScript, agnóstica de runner entre Jest, Vitest, Mocha+Chai, Jasmine, AVA,
node:test, Cypress, Playwright e Testing Library. Cobre `.js`, `.jsx`, `.ts`, `.tsx`,
`.mjs`, `.cjs`, `.mts`, `.cts`.

Os códigos compartilham um id com [Python](python.md) onde o conceito coincide; códigos `JS*` são
específicos do ecossistema. Confiança: **ALTO** bloqueia, **BAIXO** avisa. Julgamentos são
[J1-J6](../concepts/judgments.md).

## Códigos compartilhados (mesmo conceito do Python)

Estes espelham as entradas do [catálogo Python](python.md); o sinal é a forma JavaScript.

| Código | Conf | Forma JavaScript |
|---|---|---|
| C2 | ALTO | corpo de teste vazio |
| C2b | BAIXO | chama a unidade mas nunca afirma |
| C5 | ALTO | sempre verdadeira (`expect(true).toBe(true)`, `assert(1)`) |
| C6 | BAIXO | verificação fraca (`toBeTruthy`/`toBeDefined`, `.length > 0`) |
| C7 | ALTO | autocomparação (`expect(x).toBe(x)`) |
| C8 | BAIXO | igualdade exata em um float |
| C8b | BAIXO | `toBeCloseTo` sem argumento de precisão — a tolerância padrão de 2 dígitos pode ser frouxa demais |
| C9 | BAIXO | `toThrow()` sem tipo de erro ou mensagem |
| C11a | BAIXO | literal autoconfirmante — o valor esperado é ligado da mesma chamada sob teste (`const e = foo(); expect(foo()).toBe(e)`) |
| C16 | BAIXO | depende de `Date.now` / `new Date()` / `Math.random` / `crypto.randomUUID`/`getRandomValues` / um timer fixo |
| C18 | BAIXO | igualdade de string (`String(x)` / `JSON.stringify` / template literal) |
| C20 | ALTO | asserção morta depois de `return` / `throw` / `process.exit` / um `switch` exaustivo (alcançabilidade estruturada no nível de bloco, cfg.ts) |
| C21 | BAIXO | nenhuma asserção roda incondicionalmente; uma asserção morta não mascara isso (mesma alcançabilidade do cfg.ts) |
| C23 | BAIXO | lê um arquivo real em um caminho literal / URL fixa no código |
| C37 | BAIXO | caso duplicado de `it.each` / `test.each` |
| C44 | ALTO | tautologia numérica sobre um comprimento (`expect(x.length).toBeGreaterThanOrEqual(0)`) |
| C48 | BAIXO | dark patch: vira um flag de modo de teste (`process.env.NODE_ENV = "test"`, `process.env.TESTING = "1"`, `settings.TESTING = true`) e depois afirma (paridade com o `C48` do Python) |
| CC | BAIXO | asserção comentada |

## Códigos específicos de JavaScript

### JS1 - teste focado pula o resto da suíte
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

### JS2 - expect sem matcher
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

### JS3 - o snapshot é a única asserção
`J4` · BAIXO · F3

Um teste cuja única verificação é `toMatchSnapshot()` / `toMatchInlineSnapshot()`. A baseline é
gerada a partir da saída, então detecta mudança, não correção.

### JS4 - teste pulado
`J1` · BAIXO · F5

`it.skip` / `xit` / `it.todo` deixado no lugar. Excluído em silêncio da execução.

### JS5 - query ou evento async não aguardado
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

### JS6 - describe / suite vazio
`J1` · ALTO · F1

Um bloco `describe`/`suite` sem teste dentro. Não contribui com nada e pode ser lido como cobertura.

### JS7 - asserção em um setTimeout / callback then não aguardado
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

### JS8 - faz mock da unidade sob teste e a afirma diretamente
`J3` · BAIXO · F4

O teste faz mock da própria função que afirma testar, depois afirma o valor do mock. Ele testa a
configuração do mock, não o código. (A forma semântica, mais profunda, é o caso 10 / `S12`.)

### JS9 - asserção em um ramo literal morto
`J1` · ALTO · F2

`if (false) { expect(...) }` ou o `else` de `if (true)`. A asserção é inalcançável.

### JS11 - try/catch engole a asserção
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

### JS13 - query usada como instrução solta, nunca afirmada
`J4` · BAIXO · F1

`getBy*` / `queryBy*` chamado como instrução nua. A query roda mas nada é verificado nela.

### JS15 - comparação envolvida em um booleano
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

### JS17 - bloco de teste comentado
`J1` · BAIXO · F2

`// it('...', ...)` deixado no arquivo. O teste não roda.

### JS18 - callback done em vez de async/await
`J1` · BAIXO · F2

Um teste com callback `done` onde `done()` precede ou fica no mesmo nível da asserção, então o
runner conclui antes de a asserção lançar.

### JS21 - matcher referenciado mas nunca chamado
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

### JS22 - tabela it.each / test.each vazia
`J1` · ALTO · F5

`it.each([])(...)`. Zero casos são gerados, então o teste nunca roda. Paralelo de `C45`.

### JS23 - expect.assertions(N) que o corpo não consegue satisfazer
`J1` · ALTO · F5

`expect.assertions(N)` com um `N` numérico maior que as chamadas `expect()` incondicionais,
alcançáveis e não-aninhadas que podem rodar. A garantia que o autor escreveu para se proteger de
uma asserção async perdida em silêncio já nasce morta. Dispara só num déficit provável de `N`
literal: um `expect` em loop, em ramo, num `.then`/callback, ou num helper torna a contagem
indeterminada e suprime o achado. `expect.hasAssertions()` não tem contagem e é ignorado.

### JS24 - query do Cypress sem asserção
`J4` · BAIXO · F4

Uma cadeia de query do Cypress (`cy.get`/`cy.find`/`cy.contains`) usada como statement sem
`.should`/`.and` no fim e sem `expect` dentro de um callback `.then`. A query produz um subject que
nunca é asserido, o análogo cy.* da JS13. Comandos de ação (`click`/`type`/`visit`/...) fazem
trabalho em vez de consultar, então uma cadeia terminada num deles fica limpa, assim como uma
terminada em `.should`/`.and`.

### JS25 - a única asserção fica dentro de um callback de iterador de array
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

### JS26 - fake timers instalados mas nunca avançados
`J1` · BAIXO · F2

`jest.useFakeTimers()` / `vi.useFakeTimers()` (ou fake timers do `sinon`) é configurado, mas o
relógio nunca é avançado (`runAllTimers`, `advanceTimersByTime`, `tick`). O callback agendado nunca
dispara, então a asserção lê estado não-modificado e passa pelo motivo errado.

### JS27 - toHaveBeenCalled* é o único oráculo sobre um dublê criado localmente
`J3` · BAIXO · F4

A única asserção é `toHaveBeenCalled` / `toHaveBeenCalledWith` / `toHaveBeenCalledTimes` sobre um
mock criado no teste (`jest.fn()` / `vi.fn()`). Ela verifica a fiação do próprio teste, que o
dublê foi chamado, não que a unidade produziu o resultado certo.

### JS29 - cadeia resolves / rejects como instrução solta
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

### JS30 - asserção literal contra literal
`J2` · ALTO · F3

Ambos os operandos são fixos em tempo de parse: `expect(2).toBe(3)`, chai `expect(1).to.equal(1)`.
A asserção não toca a unidade sob teste, então é sempre-verdadeira ou sempre-falsa por construção,
nunca uma verificação do comportamento real.

### JS31 - try/catch engole um possível throw sem asserção sobre a exceção
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

## Códigos de diagnóstico (opcionais, OFF por padrão)

Família F8: não é false-green. `D1` roleta de asserções, `D3` assert duplicado, `D4` casos de `it.each`
sem título, `D6` `console.*` em um teste, `D7` teste anônimo, `D8` número mágico em uma
asserção, `M2` corpo de teste longo demais.

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
