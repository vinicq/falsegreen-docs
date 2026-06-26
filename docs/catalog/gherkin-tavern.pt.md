# Gherkin e Tavern

Dois formatos de DSL que a [skill](../scanners/skill.md) cobre como uma passagem semântica, baseada em texto. Os
arquivos `.feature` e `*.tavern.yaml` não são código, então os scanners estáticos não os parseiam; a
skill lê o cenário ou estágio e mapeia cada achado para [J1-J6](../concepts/judgments.md).

## Gherkin / BDD (`.feature`)

Usado por Cucumber.js, behave, pytest-bdd, SpecFlow. As definições de passo vivem em `.js`/`.ts`/`.py`
e são cobertas pelos scanners estáticos; esta passagem é sobre os cenários em si. O passo `Then`
é o oráculo.

| Padrão | J | O que detecta | Espelha |
|---|---|---|---|
| Cenário sem `Then` | J1 | só passos `Given`/`When`; nunca declara um resultado esperado | C2b |
| `Then` que não verifica | J4 | a definição de passo só age ou loga, nunca afirma | C2b |
| `Scenario Outline` vazio | J1 | uma tabela `Examples:` sem linhas roda zero vezes | C2 |
| `Then` tautológico | J2 | afirma uma constante (`Then 1 equals 1`) | C5 |

=== "RUIM"
    ```gherkin
    Scenario: User logs in
      Given a registered user
      When they submit valid credentials
      # no Then - nothing is verified
    ```
=== "LIMPO"
    ```gherkin
    Scenario: User logs in
      Given a registered user
      When they submit valid credentials
      Then they see the dashboard
    ```

**Parecidos - NÃO sinalizar:** um `Then` cuja definição de passo de fato afirma (presença de UI conta na
camada E2E); cenários marcados com `@skip`/`@wip`/`@manual` (intencionalmente não rodados, reporte como
pulados, baixo).

## Tavern (`*.tavern.yaml`)

Um plugin de pytest para testes de API HTTP/MQTT. O teste é YAML; o bloco `response:` é o oráculo.

| Padrão | J | O que detecta | Espelha |
|---|---|---|---|
| `request:` sem `response:` | J1/J4 | a chamada é enviada mas nada é verificado | C2b |
| `response:` verifica só `status_code` | J4 | "algo voltou" quando o corpo é o que importa | C6 |
| Aceitação de status ampla demais | J4 | uma lista/range de `status_code` que aceita quase qualquer resultado | C6 |
| `verify_response_with` que não afirma | J3/J4 | uma função externa que não verifica | C2b |

=== "RUIM"
    ```yaml
    stages:
      - name: create user
        request:
          url: http://localhost/users
          method: POST
        # no response - the stage passes as long as the request does not error
    ```
=== "LIMPO"
    ```yaml
    stages:
      - name: create user
        request:
          url: http://localhost/users
          method: POST
        response:
          status_code: 201
          json:
            name: Alice
    ```

**Parecidos - NÃO sinalizar:** um estágio só de setup seguido de um estágio posterior que valida a
resposta; um `response:` com um match de corpo `json:` ou validação de schema `$ext` (um oráculo real).

## Testes visuais

Uma nota que vale entre runners: um teste cuja única verificação é `percySnapshot()` /
`cy.percySnapshot()` não verifica nada localmente - o diff é aprovado por um humano em um dashboard,
de forma assíncrona - então é lido como sem-verificação (equivalente a C2b). Uma baseline de
`toHaveScreenshot()` de Playwright/Storybook é gerada a partir da saída, então se for a única asserção é
só-snapshot (JS3 / C14). Um snapshot visual *ao lado* de uma asserção comportamental está ok.
