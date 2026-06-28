# Catálogo Robot Framework

Os códigos implementados por [robotframework-falsegreen](../scanners/robot.md): uma varredura estática sobre o
parser oficial do Robot Framework (`robot.api.get_model`), sem execução. Um arquivo `.robot` é uma DSL,
então esta é uma passagem baseada em modelo que mapeia cada achado para [J1-J6](../concepts/judgments.md).

Os códigos compartilham um id com [Python](python.md) onde o conceito coincide; códigos `R*` são
específicos do Robot.

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

| Código | Conf | Forma Robot |
|---|---|---|
| C2 | ALTO | caso de teste vazio (só settings, sem keywords de corpo) / user keyword vazia |
| C2b | BAIXO | roda keywords mas nenhuma keyword de verificação |
| C3 | ALTO | falha engolida: `Run Keyword And Ignore Error` / `Return Status` não afirmado, ou um `TRY/EXCEPT` que engole |
| C5 | ALTO | sempre verdadeira (`Should Be True    ${TRUE}`) |
| C7 | ALTO | autocomparação (`Should Be Equal    ${x}    ${x}`) |
| C9 | BAIXO | erro esperado pega-tudo (`Run Keyword And Expect Error    *`) |
| C16 | BAIXO | `Sleep` como sincronização em vez de `Wait Until *`, `Get Current Date` (leitura de relógio), `Generate Random String` (aleatoriedade), ou `Evaluate` com `datetime`/`random`/`uuid` |
| C20 | ALTO | verificação depois de um terminador (`[Return]`, `Return From Keyword`, `Fail`, `Pass Execution`) |
| C21 | BAIXO | verificação só dentro de `IF` / `Run Keyword If` que pode não rodar |
| C23 | BAIXO | URL com endereço IP fixo nos dados de teste |
| C32 | BAIXO | pulado (`[Tags]    robot:skip` / `Skip`) |
| C37 | BAIXO | linha de dados de `[Template]` duplicada |
| CC | BAIXO | keyword de verificação comentada |

## Códigos específicos do Robot

### R1 - verde forçado
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

### R2 - keyword verificadora oca
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

### R3 - casos de teste em um arquivo .resource
`J1` · ALTO · F5

Uma seção `*** Test Cases ***` em um arquivo `.resource` é inválida; os casos nunca rodam.

### R4 - só No Operation
`J1` · ALTO · F1

O único passo é `No Operation` - o teste roda mas não faz nada.

### R5 - [Template] vazio
`J1` · ALTO · F5

Uma keyword `[Template]` sem linhas de dados gera zero casos. Paralelo de `C45` / `JS22`.

### R6 - Should Be True sobre um literal de string
`J4` · ALTO · F3

`Should Be True    some text` passa uma string não-vazia, que é sempre truthy, então a verificação
nunca falha. Passe uma expressão real (`${x} > 0`).

### R7 - keyword de template oca
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

## Códigos de diagnóstico (opcionais, OFF por padrão)

`D2` (controle de fluxo em um teste) e `M2` (teste longo demais) - higiene, não false-green. O Robocop também
cobre estes.

## Parecidos: NÃO sinalizar

- `Run Keyword And Expect Error` com uma mensagem/padrão ESPECÍFICO: está afirmando.
- `Wait Until Keyword Succeeds`: um retry legítimo para flakiness de E2E, não um smell de Sleep.
- Keywords de teardown (`[Teardown]`, `Close Browser`): limpeza, não o oráculo.
- Keywords de presença em E2E (`Page Should Contain Element`): a asserção na camada do browser.
- `Run Keywords    Click    AND    Should Be Equal ...`: a cadeia AND contém uma verificação real; não
  C2b.
