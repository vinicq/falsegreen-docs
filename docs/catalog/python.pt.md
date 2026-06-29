# Catálogo Python

Os códigos estruturais `C*`, como implementados pelo scanner [falsegreen](../scanners/python.md): uma
passagem de AST sem dependências sobre pytest e unittest. Cada código nomeia o sinal que dispara a regra e,
onde ajuda, o parecido que ele deliberadamente deixa em paz.

Confiança: **ALTO** bloqueia, **BAIXO** avisa, **OFF** é só diagnóstico. Julgamentos são
[J1-J6](../concepts/judgments.md); famílias são [F1-F8](../concepts/taxonomy.md).

Cada código emitido tem sua própria entrada abaixo, agrupada por família. O índice leva direto a
cada uma.

## Índice

| Código | Conf | J | Resumo |
|---|---|---|---|
| [C1](#c1) | BAIXO | J1 | assert dentro de if/for que pode nunca rodar |
| [C2](#c2) | ALTO | J1 | nenhuma verificação (corpo vazio) |
| [C2b](#c2b) | BAIXO | J1 | chama coisas mas não verifica nada |
| [C2c](#c2c) | BAIXO | J1 | bloco `self.subTest(...)` vazio |
| [C3](#c3) | ALTO | J1 | assert dentro de um try cujo except engole o erro |
| [C4](#c4) | ALTO | J1 | não coletado pelo pytest (nunca roda) |
| [C4b](#c4b) | BAIXO | J1 | classe de teste tem `__init__` (não coletada) |
| [C5](#c5) | ALTO | J2 | verificação sempre verdadeira (`assert True` / tupla / `or True`) |
| [C6](#c6) | BAIXO | J4 | verificação fraca (só que algo voltou) |
| [C6b](#c6b) | BAIXO | J5 | acoplada ao layout posicional de argumentos |
| [C6c](#c6c) | BAIXO | J4 | `call_count` como oráculo de truthiness |
| [C7](#c7) | ALTO | J2 | compara algo consigo mesmo |
| [C8](#c8) | BAIXO | J4 | igualdade exata em float |
| [C8b](#c8b) | BAIXO | J4 | igualdade aproximada sem tolerância explícita |
| [C9](#c9) | BAIXO | J4 | `pytest.raises` amplo demais |
| [C11a](#c11a) | BAIXO | J2 | literal autoconfirmante atribuído pelo teste |
| [C13](#c13) | ALTO | J3 | asserção de mock escrita errada / não chamada |
| [C13b](#c13b) | BAIXO | J3 | `patch()` sem autospec |
| [C14](#c14) | BAIXO | J2 | golden/snapshot gerado a partir da própria saída |
| [C16](#c16) | BAIXO | J1 | depende de tempo, aleatoriedade ou um sleep fixo |
| [C17](#c17) | ALTO | J1 | skip dentro de um except amplo esconde a falha |
| [C18](#c18) | BAIXO | J2 | compara `str()`/`repr()` com um literal |
| [C19](#c19) | BAIXO | J1 | `pytest.raises` envolve mais de uma chamada |
| [C20](#c20) | ALTO | J1 | asserção em código morto após return/raise/fail |
| [C21](#c21) | BAIXO | J1 | toda asserção é condicional, nenhuma roda |
| [C22](#c22) | OFF | J1 | teste async afirma mas nunca dá await na unidade |
| [C23](#c23) | BAIXO | J6 | abre um arquivo real num caminho literal |
| [C24](#c24) | BAIXO | J6 | estado mutável global compartilhado entre testes |
| [C25](#c25) | BAIXO | J4 | xfail sem `strict=True` (XPASS tratado como pass) |
| [C27](#c27) | ALTO | J1 | try/except/pass sem nenhuma asserção |
| [C28](#c28) | BAIXO | J4 | binding de `pytest.raises` nunca inspecionado |
| [C29](#c29) | BAIXO | J6 | `os.environ` atribuído direto (vaza entre testes) |
| [C30](#c30) | BAIXO | J3 | interceptor HTTP registrado mas não ativado |
| [C31](#c31) | BAIXO | J4 | saída de `capsys`/`capfd` capturada mas não afirmada |
| [C32](#c32) | BAIXO | J1 | `@pytest.mark.skip` sem `reason=` |
| [C33](#c33) | BAIXO | J4 | métrica sklearn computada mas não afirmada |
| [C34](#c34) | BAIXO | J4 | forma de assert subótima |
| [C35](#c35) | BAIXO | J1 | decorator de retry/flaky mascara flakiness |
| [C36](#c36) | BAIXO | J4 | `pytest.fail()` sem motivo |
| [C37](#c37) | BAIXO | J4 | caso de `parametrize` duplicado |
| [C38](#c38) | ALTO | J1 | dois testes com o mesmo nome (o primeiro nunca roda) |
| [C39](#c39) | ALTO | J1 | retorna uma comparação em vez de afirmá-la |
| [C41](#c41) | BAIXO | J4 | asserção sobre um mutador in-place que retorna None |
| [C42](#c42) | ALTO | J2 | asserção sobre uma generator expression ou lambda |
| [C43](#c43) | BAIXO | J1 | `pytest.skip()` depois da lógica do teste |
| [C44](#c44) | ALTO | J2 | tautologia numérica (`len()`/`abs()` sempre verdadeiro) |
| [C45](#c45) | ALTO | J1 | lista de `parametrize` vazia (zero casos) |
| [C48](#c48) | BAIXO | J1 | dark patch: liga uma flag de modo-teste e afirma |
| [C49](#c49) | BAIXO | J1 | `pytest.warns`/`assertWarns` envolve mais de uma chamada |
| [C50](#c50) | BAIXO | J4 | `caplog`/`assertLogs` capturado mas não afirmado |
| [C51](#c51) | ALTO | J1 | contexto `pytest.raises`/`warns` de corpo vazio |
| [C52](#c52) | BAIXO | J2 | autoconfirmação de pertinência (`x in {x}`) |
| [C55](#c55) | BAIXO | J3 | compara dois valores enraizados em mock |
| [C56](#c56) | BAIXO | J1 | assert síncrono de uma corrotina nunca aguardada |
| [C57](#c57) | BAIXO | J3 | compara contra um atributo de Mock não configurado |
| [C59](#c59) | ALTO | J1 | comparação solta no nível do teste (irmã loose-statement do C39) |
| [CC](#cc) | BAIXO | J1 | assert comentado |
| [D1](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J4 | roleta de asserções |
| [D3](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J4 | assert duplicado |
| [D4](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J4 | `parametrize` sem nomes |
| [D5](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J5 | setup inline excessivo |
| [D6](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J4 | `print()` de debug no corpo |
| [M2](#codigos-de-diagnostico-opcionais-off-por-padrao) | OFF | J5 | método de teste longo |
| [PL1](#pl1) | BAIXO | J1 | `-O`/`PYTHONOPTIMIZE` remove todo assert |
| [PL2](#pl2) | BAIXO | J1 | `filterwarnings` não promove avisos a erro |
| [PL7](#pl7) | BAIXO | J5 | sem gate de cobertura |
| [PL8](#pl8) | BAIXO | J5 | `addopts` interrompe a execução cedo |

Um código mantém seu id entre linguagens onde o smell é o mesmo. Dois ids carregam um sentido
diferente por linguagem, documentado em cada página: **C31** é a captura de `capsys`/`capfd` aqui,
mas o valor capturado e nunca usado (`${x}= Get Text`) no [Robot](robot.md#c31); **C44** é a
tautologia numérica aqui, ampliada no Robot para qualquer asserção de biblioteca vazia (mesmo id,
balde mais largo).

---

## Família A - o teste nunca verifica nada

Modos de falha F1 (sem oráculo) e F2 (a verificação nunca roda).

### C1 - asserção dentro de um condicional ou loop que pode nunca rodar { #c1 }
`J1` · BAIXO · F2

O `assert` (ou `self.assert*`) vive dentro de um `if`, `for` ou `while` cuja condição poderia
ser falsa ou cujo iterável poderia estar vazio. O teste passa vacuamente quando o ramo nunca é
acessado.

!!! note "Sinal"
    A asserção não é alcançável a partir do nível superior da função sem entrar em um condicional.
    Não sinalizado quando o loop itera um literal não-vazio (`for x in (1, 2, 3):`).

=== "RUIM"
    ```python
    def test_items():
        for item in items:        # items pode ser []
            assert item.valid     # nunca roda se items estiver vazio
    ```
=== "LIMPO"
    ```python
    def test_items():
        assert len(items) > 0
        for item in items:
            assert item.valid
    ```

### C2 - o corpo do teste não contém nenhuma asserção { #c2 }
`J1` · ALTO · F1

Nenhum `assert`, nenhum `self.assert*`, nenhum `pytest.raises()`, nenhum `.should.` fluente, nenhuma asserção de mock.
O corpo é só `pass`, uma docstring, `...` ou setup. Sempre verde, não importa o código.

!!! note "Sinal"
    Nenhuma verificação de qualquer tipo no corpo. Exceções: `@pytest.mark.skip`,
    `@pytest.mark.xfail` e decoradores `@hypothesis` / `@given` / `@fuzz`.

=== "RUIM"
    ```python
    def test_create_user():
        user = create_user("Alice")   # sem assert - sempre verde
    ```

### C2b - o teste chama código de produção mas não verifica nada { #c2b }
`J1` · BAIXO · F1

Como C2, mas com chamadas reais à unidade sob teste. A verificação simplesmente está faltando. Mantido separado
porque é fácil confundir com um padrão de delegação.

!!! note "Sinal"
    Uma chamada real ao SUT sem asserção depois. Exceção: se o teste chama um helper que
    contém a asserção, a verificação executa pelo helper, então não é sinalizado.

=== "RUIM"
    ```python
    def test_process():
        result = process(data)        # chama o SUT mas nenhum assert vem depois
    ```

### C2c - bloco self.subTest vazio { #c2c }
`J1` · BAIXO · F1

Um bloco `with self.subTest(...):` do unittest que envolve trabalho mas não tem nenhuma asserção
dentro - o análogo do teste vazio para subTest, já que cada sub-caso gerado roda e não verifica
nada. Mais específico que a C2b, que ele substitui nessa forma. Um subTest que asserta, levanta
exceção, ou delega para um helper `check_*`/`verify_*` não é sinalizado; o receptor tem que ser `self`/`cls`.

=== "RUIM"
    ```python
    for i in cases:
        with self.subTest(i=i):
            do_thing(i)               # nenhuma asserção dentro do bloco
    ```

### C3 - assert dentro de um try cujo except engole o erro { #c3 }
`J1` · ALTO · F2

Um `try` contém um `assert`, e o `except` captura `AssertionError`, `Exception` ou `except:`
nu com um corpo que é só `pass` / `continue`. A falha é comida; o teste fica
verde.

!!! note "Sinal"
    Asserção dentro de `try`, o handler a engole. Um handler que relança ou faz trabalho
    significativo não é C3.

=== "RUIM"
    ```python
    def test_value():
        try:
            assert compute() == 42
        except Exception:
            pass                      # C3 - esconde a falha
    ```

### C4 - função de teste não coletada pelo pytest { #c4 }
`J1` · ALTO · F5

Um `def test_*` definido aninhado dentro de outra função ou método de classe, com uma asserção real,
nunca chamado e nunca decorado como rota ou callback. O pytest só coleta testes de nível superior ou de
método de classe; este fica invisível para o runner.

!!! note "Sinal"
    `test_*` aninhado com uma asserção, sem chamador. Exceção: callbacks de framework (`@app.get`,
    `@click.command`, corrotinas aguardadas, handlers de rota) não são C4.

### C4b - a classe de teste tem `__init__` { #c4b }
`J1` · BAIXO · F5

Uma classe chamada `Test*` (ou uma subclasse de `unittest.TestCase`) define `__init__`. O pytest pula tais
classes por inteiro, então nenhum dos seus testes roda.

### C20 - asserção depois de um return / raise / fail incondicional { #c20 }
`J1` · ALTO · F2

Um `assert` aparece depois de um `return`, `raise`, `break`, `continue` ou `pytest.fail()` no
mesmo bloco. Código morto; nunca alcançado. A detecção usa alcançabilidade estruturada intra-teste
(no nível de bloco), então pega uma asserção depois de um return / raise / fail em qualquer bloco,
não só no nível superior.

=== "RUIM"
    ```python
    def test_flag():
        if not flag:
            return
        assert flag          # alcançável, ok
        return               # return incondicional
        assert True          # C20 - morto, nunca roda
    ```

### C21 - toda asserção está dentro de um condicional; nenhuma roda incondicionalmente { #c21 }
`J1` · BAIXO · F2

A função tem asserções, mas toda verificação está dentro de um ramo `if` sem um if/else
exaustivo que garanta que ao menos uma rode. O teste pode passar sem verificar nada. O mesmo modelo
de alcançabilidade estruturada (no nível de bloco) decide isso, então o C21 dispara só quando
nenhuma asserção fica na espinha garantida do teste.

### C22 - teste async que nunca aguarda a unidade sob teste { #c22 }
`J1` · OFF · F2

Um `async def test_*` faz chamadas e tem asserções mas não contém nenhum `await`, `async with`,
`async for`, e não aciona um loop (`asyncio.run`, `run_until_complete`, `anyio.run`). A
corrotina pode retornar antes de qualquer I/O completar. Opcional.

### CC - assert comentado { #cc }
`J1` · BAIXO · F2

Uma linha no corpo é `# assert ...`: uma verificação que foi comentada e deixada. A asserção
nunca roda. Um sinal forte de que o teste foi enfraquecido.

=== "RUIM"
    ```python
    def test_total():
        result = total(items)
        # assert result == 42    # CC - esta verificação está desativada
    ```

---

## Família B - a verificação é fraca ou sempre verdadeira

Em sua maioria F3 (a verificação é trivialmente verdadeira).

### C5 - asserção sempre verdadeira { #c5 }
`J2` · ALTO · F3

A asserção é estruturalmente garantida de passar: `assert True`, `assert (x, y)` (uma tupla
não-vazia é sempre truthy), `assert 1`, `assert x or True`. A verificação não adiciona proteção.

=== "RUIM"
    ```python
    def test_items():
        assert (item_a, item_b)   # C5 - tupla não-vazia, sempre True
    ```

### C6 - asserção fraca: só verifica que algo voltou { #c6 }
`J4` · BAIXO · F4

A asserção verifica só truthiness (`assert result`), comprimento não-vazio ou conteúdo de string
sem verificar o valor ou a estrutura real.

!!! note "Sinal"
    Verificação só de truthiness ou comprimento. Exceção: em testes web/browser, uma resposta truthy ou
    um locator É o contrato. `assert response.status_code` em um teste HTTP não é sinalizado.

=== "RUIM"
    ```python
    def test_users():
        result = get_users()
        assert result            # C6 - só verifica não-vazio
    ```
=== "LIMPO"
    ```python
    def test_users():
        result = get_users()
        assert len(result) == 3
        assert result[0].name == "Alice"
    ```

### C6b - asserção sobre um argumento posicional de mock via índice calculado { #c6b }
`J3` · BAIXO · F4

O teste lê `mock.call_args.args[idx]` ou `mock.call_args[0][idx]` onde `idx` é calculado
(`.index()`, aritmética, uma variável) em vez de um literal fixo. A posição é frágil e
pode mudar em silêncio.

### C6c - usar a veracidade do call_count do mock como oráculo { #c6c }
`J4` · BAIXO · F4

`assert mock.call_count` (puro) passa para qualquer contagem `>= 1`, então só verifica que o mock
foi chamado, não quantas vezes. O receptor tem que ser um mock conhecido; uma contagem exata ou com
limite inferior (`== N`, `>= 1`) é uma verificação real. A forma sempre verdadeira `mock.call_count >= 0` é a C44.

### C7 - autocomparação: ambos os lados são idênticos { #c7 }
`J2` · ALTO · F3

`assert x == x`, `assertEqual(x, x)`, ou qualquer comparação onde ambos os lados são sintaticamente
idênticos e não contêm chamadas de função. Sempre verdadeira por reflexividade.

!!! note "Sinal"
    Operandos idênticos, sem chamadas. Exceção: se o teste também verifica `x != peer`, `x in {x}`
    ou `hash(x)`, ele está testando a semântica de `__eq__` / `__hash__`, não C7.

=== "RUIM"
    ```python
    def test_name():
        name = get_name()
        assert name == name    # C7 - sempre verdadeira
    ```

### C8 - igualdade exata de float { #c8 }
`J4` · BAIXO · F4

`==` contra um literal float não-sentinela (qualquer coisa além de `0.0` ou `1.0`). A aritmética
de ponto flutuante torna a igualdade exata pouco confiável.

=== "RUIM"
    ```python
    assert compute() == 3.14159    # C8
    ```
=== "LIMPO"
    ```python
    assert compute() == pytest.approx(3.14159, rel=1e-6)
    ```

### C8b - igualdade aproximada sem tolerância explícita { #c8b }
`J4` · BAIXO · F4

`assertAlmostEqual`/`assertNotAlmostEqual` (padrão de 7 casas) ou `== pytest.approx(...)` (padrão
1e-6 relativo) sem `places=`/`delta=`/`rel=`/`abs=`. A tolerância padrão pode aprovar um valor
errado de verdade. Dimensionar a tolerância aos valores mantém quieto.

=== "RUIM"
    ```python
    self.assertAlmostEqual(total(), 4.2)      # padrão de 7 casas
    assert total() == pytest.approx(4.2)      # padrão 1e-6 rel
    ```
=== "LIMPO"
    ```python
    self.assertAlmostEqual(total(), 4.2, places=2)
    ```

### C9 - pytest.raises ampla demais { #c9 }
`J4` · BAIXO · F4

`pytest.raises()` sem tipo de exceção, ou com um muito amplo (`Exception`, `BaseException`) e
sem `match=`. Qualquer exceção, inclusive uma de um erro de digitação dentro do teste, satisfaz a verificação.

=== "RUIM"
    ```python
    with pytest.raises(Exception):   # C9 - qualquer coisa passa
        divide(a, b)
    ```
=== "LIMPO"
    ```python
    with pytest.raises(ZeroDivisionError, match="division by zero"):
        divide(a, 0)
    ```

### C11a - literal autoconfirmante: atribui e depois afirma o mesmo valor { #c11a }
`J2` · BAIXO · F3

`obj.attr = VALUE` seguido de `assert obj.attr == VALUE` com o mesmo literal. O teste
confirma que a atribuição de atributo do Python funciona, não o código de produção.

=== "RUIM"
    ```python
    def test_price():
        product.price = 100
        assert product.price == 100   # C11a - só confirma a atribuição
    ```

### C52 - autoconfirmação de pertinência { #c52 }
`J2` · BAIXO · F3

`assert x in {x}` (ou `x in [x]`, `x in (x,)`): a coleção é construída a partir do próprio
sujeito sob teste, então a pertinência vale por construção. Uma variante de pertinência da C7.
Verificar contra uma coleção montada de forma independente do sujeito é uma verificação real.

=== "RUIM"
    ```python
    def test_tag():
        tag = get_tag()
        assert tag in {tag}          # C52 - verdadeira por construcao
    ```

### C13 - asserção de mock escrita errado ou não chamada { #c13 }
`J4` · ALTO · F2

Uma asserção de mock acessada como atributo sem `()`: `mock.assert_called_once` em vez de
`mock.assert_called_once_with()`. O acesso ao atributo retorna um método ligado; a verificação nunca
roda. Também sinaliza nomes inventados (`assert_called_twice`, `called_once_with`).

=== "RUIM"
    ```python
    mock_fn.assert_called_once      # C13 - faltando (), não faz nada
    ```
=== "LIMPO"
    ```python
    mock_fn.assert_called_once_with(expected_arg)
    ```

### C13b - patch() sem autospec { #c13b }
`J3` · BAIXO · F4

`@patch('module.Thing')` ou `patch.object(obj, 'method')` sem `autospec=True`, `spec=` ou
`spec_set=`. O mock aceita qualquer assinatura de chamada em silêncio; erros de digitação em nomes ou contagens de argumentos passam
sem detecção.

### C14 - golden file gerado a partir da saída real { #c14 }
`J2` · BAIXO · F3

`if not exists(golden_path): write(golden_path, actual_output)`. Na primeira execução o teste
escreve a saída atual (possivelmente errada) como o valor esperado, depois compara contra ela
para sempre.

!!! note "Sinal"
    Write-if-missing em um golden path. Exceção: em snapshot testing de browser (Playwright,
    Selenium) isso é intencional e não é sinalizado.

### C16 - o resultado depende de tempo, aleatoriedade ou sleep não controlados { #c16 }
`J6` · BAIXO · F6

`time.sleep(N)`, `datetime.now()` / `time.time()` sem `freezegun` / `time_machine`,
`random.*` sem `random.seed()`, `torch.rand*` sem `torch.manual_seed()` ou
`train_test_split` sem `random_state=`. Também sinaliza `uuid.uuid4()` / `uuid.uuid1()` /
`uuid.getnode()` e `secrets.token_*` / `secrets.randbits` / `secrets.choice`, todos qualificados
pelo módulo. Uma chamada `from uuid import uuid4` nua e o `uuid.uuid5()` determinístico não são
sinalizados.

=== "RUIM"
    ```python
    def test_expiry():
        created = datetime.now()      # C16 - não congelado
        assert is_expired(created, ttl=0) is False
    ```

### C18 - comparação de string / repr { #c18 }
`J2` · BAIXO · F4

`==` onde um dos lados é `str(x)`, `repr(x)`, `format(x, ...)` ou uma f-string, contra um literal
de string. O formato da string é um detalhe de implementação; muda sem uma mudança semântica.

=== "RUIM"
    ```python
    assert str(user) == "User(Alice, 30)"   # C18 - acopla ao formato de str()
    ```
=== "LIMPO"
    ```python
    assert user.name == "Alice" and user.age == 30
    ```

### C25 - xfail sem strict=True { #c25 }
`J1` · BAIXO · F5

`@pytest.mark.xfail` sem `strict=True`. Se o teste passa inesperadamente, o pytest reporta
`XPASS`, não uma falha. Um xfail que passa em silêncio esconde que o bug foi corrigido sem remover
a marca.

### C34 - forma de asserção subótima { #c34 }
`J4` · BAIXO · F8

`assert not x in y` (use `x not in y`), `assert len(x) == 0` (use `assert not x`),
`assert x == True` / `== False` / `== None` / `!= None` (use `is` / truthiness). Estas enfraquecem
a mensagem de erro e obscurecem a intenção.

---

## Família C - o teste verifica o próprio setup, não o programa

### C19 - pytest.raises envolve mais de uma chamada { #c19 }
`J1` · BAIXO · F4

Um bloco `with pytest.raises(E):` contém mais de uma instrução. Se a primeira lança, a segunda
nunca roda, então o teste pode estar verificando uma linha diferente da pretendida.

=== "RUIM"
    ```python
    with pytest.raises(ValueError):
        setup_data()          # esta pode lançar, não o SUT
        sut.process(data)     # C19 - alvo pretendido
    ```

### C49 - pytest.warns / assertWarns envolve mais de uma chamada { #c49 }
`J1` · BAIXO · F4

Um bloco `with pytest.warns(W):` / `assertWarns` / `deprecated_call()` contém mais de uma
instrução. Uma linha anterior não relacionada pode emitir o warning enquanto o alvo nunca emite,
então o teste passa sem exercitar o warning sob teste. A irmã de warns da C19.

### C28 - variável de ligação do pytest.raises nunca lida { #c28 }
`J4` · BAIXO · F4

`with pytest.raises(E) as exc:` onde `exc` nunca é usado depois. O tipo da exceção é
verificado mas não sua mensagem ou atributos.

=== "RUIM"
    ```python
    with pytest.raises(ValueError) as exc:   # C28 - exc nunca lido
        process(bad_input)
    ```
=== "LIMPO"
    ```python
    with pytest.raises(ValueError) as exc:
        process(bad_input)
    assert "must be positive" in str(exc.value)
    ```

### C51 - contexto pytest.raises / warns de corpo vazio { #c51 }
`J1` · ALTO · F1

`with pytest.raises(E):` (ou `pytest.warns`) cujo corpo é vazio (`pass`, `...`, um comentário).
Nenhuma chamada é feita dentro do bloco, então a chamada que deveria lançar nunca roda e o
gerenciador de contexto não tem nada a capturar. Sempre verde.

=== "RUIM"
    ```python
    with pytest.raises(ValueError):
        pass                          # C51 - nada chamado, nada lançado
    ```

### C29 - os.environ modificado diretamente em um teste { #c29 }
`J6` · BAIXO · F6

`os.environ["KEY"] = value`, `os.environ.update(...)` ou `os.putenv(...)` no corpo de um teste. A
mudança persiste entre testes no mesmo processo. Use `monkeypatch.setenv()`.

### C55 - comparação entre dois valores enraizados em mock { #c55 }
`J3` · BAIXO · F4

`assert m.foo == m.bar` onde ambos os operandos derivam do mesmo dublê de teste (um `Mock`,
`MagicMock` ou um objeto injetado por `patch`). Cada lado é o próprio valor configurado pelo
teste, então a comparação verifica os dublês entre si, não o SUT.

### C56 - assert síncrono de uma corrotina nunca aguardada { #c56 }
`J1` · BAIXO · F2

A expressão afirmada chama um `async def` local sem dar await, então o operando é um objeto
corrotina, não o seu valor. Uma corrotina é sempre truthy e nunca é igual ao valor esperado que o
autor tinha em mente; a chamada real nunca rodou. Resolvido no arquivo inteiro contra o conjunto de
nomes `async def`.

=== "RUIM"
    ```python
    async def fetch(): ...

    def test_fetch():
        assert fetch() == {"ok": True}   # C56 - afirma a corrotina, não o resultado
    ```
=== "LIMPO"
    ```python
    def test_fetch():
        assert asyncio.run(fetch()) == {"ok": True}
    ```

### C57 - asserção compara contra um atributo de Mock não configurado { #c57 }
`J3` · BAIXO · F4

Um lado da comparação é `m.attr` onde `m` é um `Mock()` / `MagicMock()` nu, sem `spec=`/`spec_set=`
e sem atribuição a `m.attr` no corpo. O acesso ao atributo cria automaticamente um Mock filho novo
e truthy, então o lado esperado é o próprio auto-mock do teste, não um valor real. Só a forma de
atributo único (`m.attr`); ambos os lados em mock é território do C55.

=== "RUIM"
    ```python
    def test_role():
        m = Mock()
        assert user.role == m.role     # C57 - m.role é um Mock criado automaticamente
    ```
=== "LIMPO"
    ```python
    def test_role():
        assert user.role == "admin"
    ```

---

## Família D - o teste depende de estado externo ou compartilhado

Em sua maioria F6 (passa ou falha por sorte ou por ordem).

### C17 - pytest.skip() dentro de um except amplo { #c17 }
`J1` · ALTO · F5

Um `try` com uma asserção, onde o `except` é amplo e chama `pytest.skip()` ou
`skipTest()`. Uma falha real dispara o skip em vez de falhar o teste. Verde mesmo quando o
SUT está quebrado.

=== "RUIM"
    ```python
    def test_api():
        try:
            assert fetch_data() == expected
        except Exception:
            pytest.skip("skipping")   # C17 - esconde falhas reais
    ```

### C23 - caminho de arquivo absoluto ou relativo ao home fixo no código { #c23 }
`J6` · BAIXO · F6

`open("/home/user/data.csv")` ou `Path("/tmp/fixture.json").read_text()`. O caminho não
existe no CI ou em outra máquina. Use `tmp_path` ou `Path(__file__).parent / "data.csv"`.

### C24 - estado mutável de nível de módulo modificado por um teste { #c24 }
`J6` · BAIXO · F6

O módulo declara um `list`, `dict` ou `set` global; um teste o modifica sem nenhuma fixture
autouse que o reinicie. A ordem dos testes decide o resultado.

=== "RUIM"
    ```python
    _cache = {}                       # mutável de nível de módulo

    def test_fill():
        _cache["key"] = "value"       # C24 - modifica estado compartilhado

    def test_read():
        assert _cache["key"] == "value"  # passa só depois de test_fill
    ```

### C27 - try/except/pass em volta de uma chamada ao SUT sem asserção { #c27 }
`J1` · ALTO · F1

Um `try` chama o SUT sem asserção, e o `except` é só `pass`. Sucesso e falha
ambos ficam verdes. Diferente de C3, que envolve um assert; C27 não tem assert nenhum.

=== "RUIM"
    ```python
    def test_process():
        try:
            process(data)      # C27 - sucesso e falha ambos -> verde
        except Exception:
            pass
    ```

### C30 - mock de HTTP não ativado { #c30 }
`J3` · BAIXO · F4

`responses.add(...)` ou `httpretty.register_uri(...)` chamado, mas o ativador
(`@responses.activate`, `responses.start()`, `httpretty.enable()`) está ausente. O HTTP real passa
adiante; o mock nunca é usado.

### C31 - resultado de capsys.readouterr() descartado { #c31 }
`J4` · BAIXO · F1

`capsys.readouterr()` chamado como expressão nua, ou atribuído a uma variável nunca lida. A
captura rodou mas nada foi verificado.

=== "RUIM"
    ```python
    def test_output(capsys):
        run()
        capsys.readouterr()          # C31 - capturado mas nunca afirmado
    ```
=== "LIMPO"
    ```python
    def test_output(capsys):
        run()
        out, _ = capsys.readouterr()
        assert out == "hello\n"
    ```

### C50 - saída de caplog / assertLogs capturada mas nunca afirmada { #c50 }
`J4` · BAIXO · F1

`caplog` é lido (`caplog.records`, `caplog.text`) ou `self.assertLogs(...)` é aberto, mas a saída
capturada nunca é afirmada: nenhuma comparação sobre os registros, mensagens ou níveis. A captura
rodou e não teve efeito no pass/fail. A irmã de logging da C31.

=== "RUIM"
    ```python
    def test_logs(caplog):
        run()
        caplog.records               # C50 - capturado mas nunca afirmado
    ```
=== "LIMPO"
    ```python
    def test_logs(caplog):
        run()
        assert "started" in caplog.text
    ```

### C32 - @pytest.mark.skip sem reason { #c32 }
`J1` · BAIXO · F5

`@pytest.mark.skip` sem `reason=`. Nenhuma explicação para o teste desativado; pode ser esquecido
para sempre.

### C35 - decorador de retry / flaky { #c35 }
`J6` · BAIXO · F6

Um decorador chamado `flaky`, `repeat`, `retry`, `rerun` ou `flake` em um teste. Mascara
não-determinismo em vez de corrigi-lo.

---

## Família E - passa, mas verifica a coisa errada

### C33 - métrica de ML calculada mas não afirmada { #c33 }
`J4` · BAIXO · F1

Uma métrica do sklearn (`accuracy_score`, `f1_score`, `model.score()`) cujo resultado é descartado ou
atribuído a uma variável nunca lida. A métrica foi calculada mas nunca validada contra um
limiar.

=== "RUIM"
    ```python
    def test_model():
        acc = accuracy_score(y_true, y_pred)   # C33 - nunca afirmado
    ```
=== "LIMPO"
    ```python
    def test_model():
        acc = accuracy_score(y_true, y_pred)
        assert acc >= 0.90
    ```

### C36 - pytest.fail() sem reason { #c36 }
`J1` · BAIXO · F8

`pytest.fail()` sem mensagem. A falha fica ininteligível na saída do CI.

### C37 - caso duplicado de parametrize { #c37 }
`J2` · BAIXO · F8

`@pytest.mark.parametrize` onde o mesmo conjunto de argumentos aparece duas vezes. A duplicata confirma o
mesmo caminho de código de novo e não adiciona cobertura.

---

## Adições à família (sync do catálogo)

### C38 - dois testes compartilham um nome { #c38 }
`J1` · ALTO · F5

Dois `def test_*` no escopo de módulo ou de classe com o mesmo nome. Python liga o posterior sobre o
anterior, então o primeiro nunca roda.

### C39 - retorna uma comparação em vez de afirmar { #c39 }
`J1` · ALTO · F1

`return x == y` em um teste. O pytest ignora o valor retornado (avisa com
`PytestReturnNotNoneWarning`); nada é verificado.

### C41 - asserção sobre um mutador que retorna None { #c41 }
`J4` · BAIXO · F3

`assert not lst.sort()` / `assertIsNone(lst.sort())`. Se é trivialmente verde depende do
tipo do receptor, então este é um julgamento só da skill, restrito a mutadores conhecidos
(`sort`, `append`, `extend`, `reverse`, `update`, `add`, `remove`, `insert`, `clear`).

### C42 - asserção sobre um generator ou lambda { #c42 }
`J2` · ALTO · F3

`assert (x for x in y)` / `assert lambda: ...`. O objeto é sempre truthy. Uma list, set ou
dict comprehension não é C42, porque pode ser vazia.

### C43 - skip no meio do teste { #c43 }
`J1` · BAIXO · F5

`pytest.skip()` depois da lógica do teste, com verificações abaixo dele que então nunca rodam. Um skip no topo é
um guard legítimo.

### C44 - tautologia numérica { #c44 }
`J2` · ALTO · F3

`len(x) >= 0`, `abs(x) >= 0`, `len(x) > -1`, ou o `call_count >= 0` / `> -1` de um mock. A
comparação é sempre verdadeira.

### C45 - parametrize vazio { #c45 }
`J1` · ALTO · F5

`@pytest.mark.parametrize("...", [])`. Zero casos são gerados, então o teste nunca roda.

### C48 - dark patch: vira um flag de modo de teste e depois afirma { #c48 }
`J1` · BAIXO · F2

O teste força um toggle de modo de teste para o modo de teste (`os.environ["TESTING"] = "1"`,
`settings.TESTING = True`, um `TESTING = True` declarado com `global`) e depois afirma, então
exercita o ramo só-de-teste do produto (`if TESTING: ...`) em vez do comportamento real.

!!! note "Sinal"
    Um flag de modo de teste conhecido virado antes da asserção. Não dispara quando uma asserção
    genuína já roda antes da virada, a menos que uma asserção pós-virada leia o próprio flag
    alterado. Valores de config e feature flags de produto não são sinalizados.

=== "RUIM"
    ```python
    def test_login():
        os.environ["TESTING"] = "1"   # C48 - força o ramo só-de-teste
        assert login(user, pwd) is True
    ```

### C59 - comparação solta no nível do teste { #c59 }
`J1` · ALTO · F1

`result == expected` escrito como instrução, não dentro de um `assert`. O Python computa a
comparação e descarta o resultado, então nada é verificado. A irmã loose-statement do C39 (que
retorna a comparação); o C59 é dono da linha para o C2b não reportar em duplicidade.

=== "RUIM"
    ```python
    def test_total():
        total(items) == 42       # C59 - computado e descartado, sem assert
    ```
=== "LIMPO"
    ```python
    def test_total():
        assert total(items) == 42
    ```

---

## Camada de projeto - auditoria de config (PL)

Emitidos pela passagem de auditoria de config, não pela varredura por arquivo. A suíte fica verde
por configuração, não por um smell dentro de algum arquivo de teste.

### PL1 - asserções removidas em tempo de execução { #pl1 }
`J1` · BAIXO · F2

`python -O` / `-OO` ou `PYTHONOPTIMIZE` no ambiente de execução remove todo `assert` na compilação,
então a suíte inteira passa sem nenhuma verificação. Rode sem `-O` e desative `PYTHONOPTIMIZE`.

### PL2 - avisos não promovidos a erro { #pl2 }
`J1` · BAIXO · F4

O `filterwarnings` do pytest não transforma avisos em erros, então deprecations e warnings de
runtime passam em silêncio. Use `filterwarnings = error` para que falhem a suíte.

### PL7 - sem gate de cobertura { #pl7 }
`J5` · BAIXO · F8

Nenhum `--cov-fail-under` / `[tool.coverage.report] fail_under` está configurado, então a cobertura
pode cair a zero e a suíte ainda passa. Adicione um limiar de cobertura.

### PL8 - a execução para cedo { #pl8 }
`J5` · BAIXO · F5

`addopts` carrega `-x` / `--maxfail` / `--exitfirst`, então a execução para nas primeiras falhas e a
contagem de testes reportada fica incompleta. Remova-os para a suíte inteira rodar.

---

## Códigos de diagnóstico (opcionais, OFF por padrão)

Família F8: não é false-green (o teste ainda protege), mostrado só numa passagem de diagnóstico. Linters
dedicados (ruff) também cobrem estes.

| Código | O que sinaliza |
|---|---|
| **D1** | roleta de asserções: duas ou mais asserções, nenhuma com mensagem |
| **D3** | assert duplicado: o `assert` idêntico aparece duas vezes |
| **D4** | parametrize sem nomes: 3+ casos sem `ids=` |
| **D5** | setup inline excessivo: mais de 5 instruções antes do primeiro assert |
| **D6** | `print()` de debug deixado no corpo do teste |
| **M2** | método de teste longo: corpo com mais de 50 linhas |

---

## Parecidos: NÃO sinalizar

Estes lembram um smell mas estão corretos. O scanner os deixa em paz.

- `@pytest.mark.skip` / `@pytest.mark.xfail` em um corpo vazio: explicitamente desativado, não C2.
- `@given` / `@hypothesis` / `@fuzz` sem `assert` explícito: o hypothesis afirma internamente,
  não C2.
- Um helper chamado pelo teste que contém o `assert`: não C2b.
- `for x in (1, 2, 3): assert x`: não C1, o literal é não-vazio.
- `assert response` em um teste HTTP / `assert locator` em um teste Playwright: não C6, a presença é
  a asserção nessa camada.
- `assert x == x` onde o teste também verifica `x != peer` ou `hash(x)`: testando `__eq__` /
  `__hash__`, não C7.
- `freezegun` / `time_machine` importado: um `datetime.now()` não congelado não é C16.
- `patch(..., autospec=True)`: não C13b.
- `with pytest.raises(E) as exc: ...; assert "msg" in str(exc.value)`: exc é lido, não C28.
