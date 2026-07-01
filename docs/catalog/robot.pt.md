# Catálogo Robot Framework

Os códigos implementados por [robotframework-falsegreen](../scanners/robot.md): uma varredura estática sobre o
parser oficial do Robot Framework (`robot.api.get_model`), sem execução. Um arquivo `.robot` é uma DSL,
então esta é uma passagem baseada em modelo que mapeia cada achado para [J1-J6](../concepts/judgments.md).

Os códigos compartilham um id com [Python](python.md) onde o conceito coincide; códigos `R*` são
específicos do Robot. Confiança: **ALTO** bloqueia, **BAIXO** avisa, **OFF** é só diagnóstico.

Cada código emitido tem sua própria entrada abaixo. O índice leva direto a cada uma.

## Índice

| Código | Conf | J | Resumo |
|---|---|---|---|
| [C2](#c2) | ALTO | J1 | teste/task/keyword vazio (nenhuma keyword roda) |
| [C2b](#c2b) | BAIXO | J1 | roda keywords mas nenhuma keyword de verificação |
| [C3](#c3) | ALTO | J1 | falha engolida (`Run Keyword And Ignore Error`, `TRY/EXCEPT`) |
| [C5](#c5) | ALTO | J2 | sempre verdadeira (`Should Be True ${TRUE}`) |
| [C6](#c6) | BAIXO | J4 | `Should Be True` numa variável nua (só truthiness) |
| [C7](#c7) | ALTO | J2 | autocomparação (`Should Be Equal ${x} ${x}`) |
| [C9](#c9) | BAIXO | J4 | erro esperado pega-tudo (`Run Keyword And Expect Error *`) |
| [C9b](#c9b) | BAIXO | J1 | `expected_status=any` da RequestsLibrary desativa o oráculo |
| [C11a](#c11a) | ALTO | J2 | literal autoconfirmante (uma cópia do real alimenta o esperado) |
| [C16](#c16) | BAIXO | J1 | `Sleep`, leitura de relógio ou aleatoriedade |
| [C20](#c20) | ALTO | J1 | verificação após um terminador (`[Return]`/`Fail`/`Pass Execution`) |
| [C21](#c21) | BAIXO | J1 | verificação só dentro de `IF`/`Run Keyword If` |
| [C23](#c23) | BAIXO | J6 | URL com endereço IP fixo nos dados de teste |
| [C31](#c31) | BAIXO | J4 | valor capturado nunca usado (`${x}= Get Text`) |
| [C32](#c32) | BAIXO | J1 | teste pulado (`robot:skip` / `Skip`) |
| [C37](#c37) | BAIXO | J4 | linha de dados de `[Template]` duplicada |
| [C44](#c44) | ALTO | J2 | asserção de biblioteca vazia (verdadeira para qualquer valor) |
| [CC](#cc) | BAIXO | J1 | keyword de verificação comentada |
| [R1](#r1) | ALTO | J1 | `Pass Execution` força verde |
| [R2](#r2) | BAIXO | J1 | keyword verificadora oca (nome de oráculo, não afirma nada) |
| [R3](#r3) | ALTO | J1 | `*** Test Cases ***` num arquivo `.resource` (nunca roda) |
| [R4](#r4) | ALTO | J1 | `No Operation` é o único passo |
| [R5](#r5) | ALTO | J1 | `[Template]` sem linhas de dados |
| [R6](#r6) | BAIXO | J4 | `Should Be True` num literal de string (sempre truthy) |
| [R7](#r7) | BAIXO | J1 | keyword de template oca (todo caso gerado fica sem oráculo) |
| [R8](#r8) | ALTO | J1 | a única verificação vive no `[Setup]` |
| [R8b](#r8b) | BAIXO | J1 | a única verificação vive no `[Teardown]` |
| [D2](#diagnostics) | OFF | J4 | controle de fluxo no nível do teste |
| [M2](#diagnostics) | OFF | J5 | teste longo demais |
| [PL9](#pl9) | BAIXO | J1 | skip-on-failure / noncritical vira uma falha em pass |

Um código mantém seu id entre linguagens onde o smell é o mesmo; o sinal abaixo é a forma Robot.
Dois ids compartilhados carregam um sentido específico do Robot, ambos documentados aqui:

- **C31** aqui é um valor capturado que nunca é usado (`${x}= Get Text locator` sem asserção
  posterior sobre `${x}`). No [Python](python.md#c31) o C31 é a captura de `capsys`/`capfd`
  descartada. Mesmo conceito (um valor capturado fica sem asserção), mecanismo diferente.
- **C44** aqui é ampliado além da tautologia numérica que ele nomeia no [Python](python.md#c44) e no
  [JS](javascript-typescript.md#c44) para qualquer asserção de biblioteca vazia
  (`Should Contain ${EMPTY}`, `Should Not Be Empty ${TRUE}`, uma tautologia de `Length Should Be`).
  Mesmo id, balde mais largo, documentado em vez de drift silencioso.

## O que conta como verificação (o oráculo)

Toda a checagem de false-green depende de reconhecer as keywords de asserção, para que uma verificação real não seja
confundida com "nenhuma verificação". A convenção dominante é a palavra **`Should`**, mais
formas específicas de biblioteca:

- **BuiltIn / Collections / String:** `Should Be Equal`, `Should Be True`, `Should Contain`,
  `Should Match`, `Length Should Be`, `List Should Contain Value`, `Dictionary Should Contain Key`.
- **SeleniumLibrary / AppiumLibrary:** `Page Should Contain*`, `Element Should Be Visible`,
  `Element Text Should Be`, `Title Should Be`. `Wait Until Page Contains` / `Wait Until Element Is
  Visible` também verificam (falham no timeout).
- **Browser (Playwright):** o motor de asserção `Get ...    <selector>    <operator>    <expected>`
  onde o operador é `==`, `!=`, `contains`, etc. Um `Get Text    h1` nu sem operador não verifica
  nada.
- **RequestsLibrary:** `Status Should Be`, `Request Should Be Successful`, e uma keyword de requisição
  (`GET`/`POST`/...) carregando `expected_status=<code>` (falha se o status diferir).
  `expected_status=any` desativa a checagem, então não conta.
- **RESTinstance:** keywords de schema (`Integer`, `Number`, `String`, `Object`, ...).
- **DatabaseLibrary:** `Row Count Should Be Equal`, `Check If (Not) Exists In Database`.
- Uma **keyword customizada** do projeto cujo nome contém `Should`/`Verify`/`Assert`/`Check` e cujo
  corpo de fato chama uma das acima.

Um teste sem nenhuma destas (só `Click`, `Go To`, `Input Text`, `Log`, um `Get *` nu) não verifica
nada.

## Códigos compartilhados (mesmo conceito do Python)

O sinal é a forma Robot; o modo de falha é o mesmo da entrada Python de id igual.

### C2 - teste, task ou keyword vazio { #c2 }
`J1` · ALTO · F1

Um caso de teste vazio (só settings, sem keywords de corpo), task ou user keyword. Nada roda.

### C2b - roda keywords mas nenhuma keyword de verificação { #c2b }
`J1` · BAIXO · F1

O corpo roda keywords mas nenhuma verifica nada (sem `Should`, sem asserção de biblioteca). Sem
oráculo.

### C3 - falha engolida { #c3 }
`J1` · ALTO · F2

`Run Keyword And Ignore Error` / `Run Keyword And Return Status` cujo status nunca é afirmado, ou um
`TRY/EXCEPT` que engole a falha. O status fica sem checagem.

### C5 - verificação sempre verdadeira { #c5 }
`J2` · ALTO · F3

`Should Be True    ${TRUE}`, `Should Be Equal` com dois literais iguais, ou um `Set Variable If`
constante-verdadeiro alimentando o lado esperado. Sempre passa.

### C6 - verificação fraca numa variável nua { #c6 }
`J4` · BAIXO · F4

`Should Be True    ${x}` numa variável nua checa só truthiness, não uma comparação. Compare o valor
com `Should Be Equal`.

=== "RUIM"
    ```robotframework
    Login Works
        ${ok}=    Login
        Should Be True    ${ok}       # C6 - só truthiness
    ```
=== "LIMPO"
    ```robotframework
    Login Works
        ${ok}=    Login
        Should Be Equal    ${ok}    ${TRUE}
    ```

### C7 - autocomparação { #c7 }
`J2` · ALTO · F3

`Should Be Equal    ${x}    ${x}`: os dois lados são a mesma variável, sempre iguais.

### C9 - erro esperado pega-tudo { #c9 }
`J4` · BAIXO · F4

`Run Keyword And Expect Error    *` (ou `GLOB:*` / `REGEXP:.*`) aceita qualquer erro, inclusive um
que o teste nunca quis disparar.

### C9b - `expected_status=any` desativa o oráculo { #c9b }
`J1` · BAIXO · F4

Um método HTTP da RequestsLibrary com `expected_status=any` / `anything`: a requisição aceita todo
status, então o oráculo fica desativado e um 500 nunca falha.

### C11a - literal autoconfirmante { #c11a }
`J2` · ALTO · F3

Uma cópia do real no corpo alimenta o lado esperado: `${y}=    Set Variable    ${x}`, depois
`Should Be Equal    ${x}    ${y}`. O oráculo confirma a si mesmo.

### C16 - fonte não determinística { #c16 }
`J1` · BAIXO · F6

`Sleep` usado como sincronização em vez de `Wait Until *`, `Get Current Date` (leitura de relógio),
`Generate Random String` (aleatoriedade), ou `Evaluate` com `datetime`/`random`/`uuid`.

### C20 - verificação depois de um terminador { #c20 }
`J1` · ALTO · F2

Uma keyword de verificação depois de `[Return]`, `Return From Keyword`, `Fail` ou `Pass Execution`
no mesmo bloco: um passo morto que nunca roda.

### C21 - verificação só roda condicionalmente { #c21 }
`J1` · BAIXO · F2

A única verificação fica dentro de um `IF` / `Run Keyword If` que pode não rodar, então o teste pode
passar sem checar nada.

### C23 - URL com endereço IP fixo { #c23 }
`J6` · BAIXO · F6

Uma URL com endereço IP fixo nos dados de teste: acoplamento de ambiente / mystery guest. Leia de
uma variável ou resource.

### C31 - valor capturado nunca usado { #c31 }
`J4` · BAIXO · F1

`${x}=    Get Text    locator` (ou uma captura parecida) cujo resultado nunca é afirmado depois. A
captura está morta; o teste verifica outra coisa. Análogo Robot do C31 do `capsys` no Python,
mecanismo diferente, mesma ideia: um valor capturado fica sem asserção.

=== "RUIM"
    ```robotframework
    Title Is Set
        ${title}=    Get Text    h1     # C31 - capturado, nunca afirmado
        Click    submit
    ```
=== "LIMPO"
    ```robotframework
    Title Is Set
        ${title}=    Get Text    h1
        Should Be Equal    ${title}    Welcome
    ```

### C32 - teste pulado { #c32 }
`J1` · BAIXO · F5

`[Tags]    robot:skip` ou a keyword `Skip` deixa o teste fora da execução.

### C37 - linha de dados de `[Template]` duplicada { #c37 }
`J4` · BAIXO · F8

A mesma linha de dados aparece duas vezes num `[Template]`: a duplicata roda o mesmo cenário e não
adiciona cobertura.

### C44 - asserção de biblioteca vazia { #c44 }
`J2` · ALTO · F3

Uma asserção de biblioteca provavelmente verdadeira para qualquer valor: `Should Contain    ${x}    ${EMPTY}`,
`Should Not Be Empty    ${TRUE}`, `Should Be Empty    ${EMPTY}`, uma tautologia de
`Length Should Be`. Ampliada a partir da forma numérica que o mesmo id nomeia no Python/JS.

### CC - keyword de verificação comentada { #cc }
`J1` · BAIXO · F2

Uma linha no corpo é uma keyword de verificação comentada (`# Should Be Equal ...`): o oráculo está
desligado.

## Códigos específicos do Robot

### R1 - verde forçado { #r1 }
`J1` · ALTO · F3

`Pass Execution` (ou `Pass Execution If` com uma condição sempre verdadeira) força o teste a passar
independentemente de qualquer verificação.

=== "RUIM"
    ```robotframework
    Login Works
        Open App
        Pass Execution    skipping for now
    ```
=== "LIMPO"
    ```robotframework
    Login Works
        Open App
        Page Should Contain    Welcome
    ```

### R2 - keyword verificadora oca { #r2 }
`J1` · BAIXO · F1

Uma user keyword nomeada como um oráculo (`Verify *`, `Assert *`, `Should *`, `Check *`) cujo corpo
não contém nenhuma keyword de verificação. Um teste chamando `Verify Login` parece protegido mas não afirma
nada - a causa raiz de um C2b perdido.

=== "RUIM"
    ```robotframework
    *** Keywords ***
    Verify Login
        Log    logged in       # no Should/assertion - hollow
    ```
=== "LIMPO"
    ```robotframework
    *** Keywords ***
    Verify Login
        Page Should Contain    Welcome
    ```

### R3 - casos de teste em um arquivo .resource { #r3 }
`J1` · ALTO · F5

Uma seção `*** Test Cases ***` em um arquivo `.resource` é inválida; os casos nunca rodam.

### R4 - só No Operation { #r4 }
`J1` · ALTO · F1

O único passo é `No Operation` - o teste roda mas não faz nada.

### R5 - [Template] vazio { #r5 }
`J1` · ALTO · F5

Uma keyword `[Template]` sem linhas de dados gera zero casos. Paralelo de `C45` / `JS22`.

### R6 - Should Be True sobre um literal de string { #r6 }
`J4` · ALTO · F3

`Should Be True    some text` passa uma string não-vazia, que é sempre truthy, então a verificação
nunca falha. Passe uma expressão real (`${x} > 0`).

### R7 - keyword de template oca { #r7 }
`J1` · BAIXO · F1

Um teste com `[Template]` cuja keyword de template é definida no mesmo arquivo e não contém
verificação: toda linha de dados passa de graça. Só sinalizado quando a keyword de template resolve
no arquivo; uma keyword de template externa/importada é deixada em paz (ela pode verificar via uma keyword que o
scanner não consegue ver).

=== "RUIM"
    ```robotframework
    *** Test Cases ***
    Check Logins
        [Template]    Do Login
        alice    secret
        bob      hunter2

    *** Keywords ***
    Do Login
        [Arguments]    ${user}    ${pass}
        Input Text    user    ${user}
        Click    submit          # no verification - every row is green
    ```
=== "LIMPO"
    ```robotframework
    *** Keywords ***
    Do Login
        [Arguments]    ${user}    ${pass}
        Input Text    user    ${user}
        Click    submit
        Page Should Contain    Welcome
    ```

### R8 - a única verificação vive no [Setup] { #r8 }
`J1` · ALTO · F2

Um teste cuja única keyword de verificação fica no `[Setup]` (ou num `Test Setup` de suíte). O
setup roda antes de o corpo agir, então verifica pré-condições, não o resultado: o corpo pode
quebrar e a suíte fica verde. Mova a asserção para o corpo.

=== "RUIM"
    ```robotframework
    Order Is Shipped
        [Setup]    Should Be Equal    ${status}    pending
        Ship Order                      # corpo age, nunca verificado
    ```
=== "LIMPO"
    ```robotframework
    Order Is Shipped
        Ship Order
        Should Be Equal    ${order.status}    shipped
    ```

### R8b - a única verificação vive no [Teardown] { #r8b }
`J1` · BAIXO · F2

A única keyword de verificação fica no `[Teardown]` (ou `Test Teardown`). O teardown roda mesmo
quando o corpo falha e reporta num eixo separado (pode trocar o motivo de um teste que falhou),
então não verifica o resultado do corpo. Confiança menor que a R8 porque uma verificação de
teardown às vezes é uma asserção de limpeza deliberada.

## Camada de projeto - config de execução (PL)

### PL9 - skip-on-failure vira uma falha em pass { #pl9 }
`J1` · BAIXO · F5

Um `skip-on-failure` / `noncritical` na config de execução transforma um teste que falha num pass
não fatal. Legado (removido no Robot Framework 4+), ainda visto em suítes antigas.

## Códigos de diagnóstico (opcionais, OFF por padrão) { #diagnostics }

Família F8: higiene, não false-green. O Robocop também cobre estes.

| Código | O que sinaliza |
|---|---|
| **D2** | controle de fluxo (`IF`/`FOR`/`WHILE`/`TRY`) no nível do teste/task |
| **M2** | teste/task longo demais (o guia sugere no máximo ~10 passos) |

## Parecidos: NÃO sinalizar

- `Run Keyword And Expect Error` com uma mensagem/padrão ESPECÍFICO: está afirmando.
- `Wait Until Keyword Succeeds`: um retry legítimo para flakiness de E2E, não um smell de Sleep.
- Keywords de teardown (`[Teardown]`, `Close Browser`): limpeza, não o oráculo.
- Keywords de presença em E2E (`Page Should Contain Element`): a asserção na camada do browser.
- `Run Keywords    Click    AND    Should Be Equal ...`: a cadeia AND contém uma verificação real; não
  C2b.
