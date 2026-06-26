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
| C9 | BAIXO | `toThrow()` sem tipo de erro ou mensagem |
| C16 | BAIXO | depende de `Date.now` / `Math.random` / um timer fixo |
| C18 | BAIXO | igualdade de string (`String(x)` / `JSON.stringify` / template literal) |
| C20 | ALTO | asserção em código morto depois de `return` / `throw` |
| C21 | BAIXO | toda asserção é condicional |
| C23 | BAIXO | lê um arquivo real em um caminho literal / URL fixa no código |
| C37 | BAIXO | caso duplicado de `it.each` / `test.each` |
| C44 | ALTO | tautologia numérica sobre um comprimento (`expect(x.length).toBeGreaterThanOrEqual(0)`) |
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
