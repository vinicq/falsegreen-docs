# Patterns by language and test level

Esta página lista o que cada scanner da família detecta, organizado de duas formas: por
linguagem (um eixo fixo) e por nível de teste (unidade, integração, E2E). As páginas do catálogo
trazem as definições completas; aqui está o mapa que liga as duas.

## An honest note about level

Unidade, integração e E2E **não são uma partição fixa por código**. O nível é uma classificação
por achado, decidida no [J3](judgments.md): o teste exercita a unidade real, um colaborador ou
borda de integração, ou o stack E2E completo. O mesmo padrão de false-green lê no nível em que o
teste opera. Então a lista abaixo é completa por linguagem (o eixo fixo), e os clusters de nível
no fim são os padrões *característicos* de cada nível, com a ressalva de que a maioria dos
códigos pode aparecer em mais de um nível.

## The model

Os julgamentos [J1-J6](judgments.md) perguntam, em ordem: a assertion roda, o oráculo é
independente, exercita a unidade real ou um dublê, verifica o suficiente, está acoplado a
internos, passa em isolamento. A [skill](../scanners/skill.md) é o superconjunto dos três
scanners estruturais: os códigos estruturais (`C*` / `JS*` / `R*` / `PL*`) mais os semânticos
(`S1`-`S21`), que precisam de uma leitura de intenção que um AST não decide.

## Python (falsegreen)

O scanner [falsegreen](../scanners/python.md) emite 67 códigos sobre pytest e unittest. Cada um
liga para sua [entrada no catálogo](../catalog/python.md).

| Code | J | Conf | What it catches |
|---|---|---|---|
| [C1](../catalog/python.md#c1) | J1 | LOW | assertion dentro de condicional ou loop que pode nunca rodar |
| [C2](../catalog/python.md#c2) | J1 | HIGH | corpo do teste sem nenhuma assertion |
| [C3](../catalog/python.md#c3) | J1 | HIGH | assert dentro de um try cujo except engole o erro |
| [C4](../catalog/python.md#c4) | J1 | HIGH | função de teste não coletada pelo pytest |
| [C5](../catalog/python.md#c5) | J2 | HIGH | assertion sempre verdadeira |
| [C6](../catalog/python.md#c6) | J4 | LOW | assertion fraca: só checa que algo voltou |
| [C7](../catalog/python.md#c7) | J2 | HIGH | autocomparação: os dois lados são idênticos |
| [C8](../catalog/python.md#c8) | J4 | LOW | igualdade exata de float |
| [C9](../catalog/python.md#c9) | J4 | LOW | `pytest.raises` amplo demais |
| [CC](../catalog/python.md#cc) | J1 | LOW | assert comentado |
| [C13](../catalog/python.md#c13) | J4 | HIGH | assertion de mock escrita errada ou não chamada |
| [C14](../catalog/python.md#c14) | J2 | LOW | golden file gerado a partir da saída real |
| [C16](../catalog/python.md#c16) | J6 | LOW | resultado depende de tempo, aleatoriedade ou sleep não controlados |
| [C17](../catalog/python.md#c17) | J1 | HIGH | `pytest.skip()` dentro de except amplo |
| [C18](../catalog/python.md#c18) | J2 | LOW | comparação de string/repr |
| [C19](../catalog/python.md#c19) | J1 | LOW | `pytest.raises` envolve mais de uma chamada |
| [C20](../catalog/python.md#c20) | J1 | HIGH | assertion após return/raise/fail incondicional |
| [C21](../catalog/python.md#c21) | J1 | LOW | toda assertion está dentro de condicional; nenhuma roda incondicionalmente |
| [C22](../catalog/python.md#c22) | J1 | OFF | teste async nunca dá await na unidade sob teste |
| [C23](../catalog/python.md#c23) | J6 | LOW | caminho de arquivo absoluto ou relativo ao home hard-coded |
| [C24](../catalog/python.md#c24) | J6 | LOW | estado mutável a nível de módulo mutado pelo teste |
| [C25](../catalog/python.md#c25) | J1 | LOW | xfail sem `strict=True` |
| [C27](../catalog/python.md#c27) | J1 | HIGH | try/except/pass em volta da chamada do SUT sem assertion |
| [C28](../catalog/python.md#c28) | J4 | LOW | variável vinculada por `pytest.raises` nunca lida |
| [C29](../catalog/python.md#c29) | J6 | LOW | `os.environ` modificado direto no teste |
| [C2b](../catalog/python.md#c2b) | J1 | LOW | chama código de produção mas não verifica nada |
| [C2c](../catalog/python.md#c2c) | J1 | LOW | bloco `self.subTest(...)` vazio |
| [C30](../catalog/python.md#c30) | J3 | LOW | mock HTTP não ativado |
| [C31](../catalog/python.md#c31) | J4 | LOW | resultado de `capsys.readouterr()` descartado |
| [C32](../catalog/python.md#c32) | J1 | LOW | `@pytest.mark.skip` sem motivo |
| [C33](../catalog/python.md#c33) | J4 | LOW | métrica de ML calculada mas não asserida |
| [C34](../catalog/python.md#c34) | J4 | LOW | forma de assertion subótima |
| [C35](../catalog/python.md#c35) | J6 | LOW | decorator de retry/flaky |
| [C36](../catalog/python.md#c36) | J1 | LOW | `pytest.fail()` sem motivo |
| [C37](../catalog/python.md#c37) | J2 | LOW | caso de parametrize duplicado |
| [C38](../catalog/python.md#c38) | J1 | HIGH | dois testes com o mesmo nome |
| [C39](../catalog/python.md#c39) | J1 | HIGH | retorna uma comparação em vez de asserir |
| [C41](../catalog/python.md#c41) | J4 | LOW | assertion sobre um mutator que retorna None |
| [C42](../catalog/python.md#c42) | J2 | HIGH | assertion sobre um generator ou lambda |
| [C43](../catalog/python.md#c43) | J1 | LOW | skip no meio do teste |
| [C44](../catalog/python.md#c44) | J2 | HIGH | tautologia numérica |
| [C45](../catalog/python.md#c45) | J1 | HIGH | parametrize vazio |
| [C48](../catalog/python.md#c48) | J1 | LOW | dark patch: liga uma flag de modo-teste e então asserta |
| [C49](../catalog/python.md#c49) | J1 | LOW | `pytest.warns`/`assertWarns` envolve mais de uma chamada |
| [C4b](../catalog/python.md#c4b) | J1 | LOW | classe de teste tem `__init__` (pytest não coleta) |
| [C50](../catalog/python.md#c50) | J4 | LOW | log capturado nunca asserido |
| [C51](../catalog/python.md#c51) | J1 | HIGH | contexto `pytest.raises`/`warns` de corpo vazio |
| [C52](../catalog/python.md#c52) | J2 | LOW | autoconfirmação de pertinência |
| [C55](../catalog/python.md#c55) | J3 | LOW | assertion compara dois valores ancorados em mock |
| [C56](../catalog/python.md#c56) | J1 | LOW | assert síncrono de uma coroutine nunca awaitada |
| [C57](../catalog/python.md#c57) | J3 | LOW | assertion contra um atributo de Mock não configurado |
| [C59](../catalog/python.md#c59) | J1 | HIGH | comparação solta escrita como statement |
| [C6b](../catalog/python.md#c6b) | J3 | LOW | assertion sobre argumento posicional de mock via índice calculado |
| [C6c](../catalog/python.md#c6c) | J4 | LOW | truthiness de `call_count` do mock como oráculo |
| [C8b](../catalog/python.md#c8b) | J4 | LOW | igualdade aproximada sem tolerância explícita |
| [C11a](../catalog/python.md#c11a) | J2 | LOW | literal autoconfirmante: o teste atribui e então asserta o mesmo valor |
| [C13b](../catalog/python.md#c13b) | J3 | LOW | `patch()` sem autospec |
| [D1](../catalog/python.md#diagnostics) | - | LOW | assertion roulette: vários asserts, nenhum com mensagem |
| [D3](../catalog/python.md#diagnostics) | - | LOW | assert duplicado: a mesma assertion aparece duas vezes |
| [D4](../catalog/python.md#diagnostics) | - | LOW | casos de parametrize sem nome |
| [D5](../catalog/python.md#diagnostics) | - | LOW | setup inline excessivo |
| [D6](../catalog/python.md#diagnostics) | - | LOW | print de debug no teste |
| [M2](../catalog/python.md#diagnostics) | - | LOW | método de teste longo |
| [PL1](../catalog/python.md#pl1) | J1 | n/a | asserts removidos em runtime |
| [PL2](../catalog/python.md#pl2) | J1 | n/a | warnings não promovidos |
| [PL7](../catalog/python.md#pl7) | J5 | n/a | sem gate de cobertura |
| [PL8](../catalog/python.md#pl8) | J5 | n/a | execução para cedo |

## JavaScript / TypeScript and the TSX/JSX family (falsegreen-js)

O scanner [falsegreen-js](../scanners/javascript.md) emite 52 códigos: os específicos de JS mais
os `C*` compartilhados. Cada um liga para sua
[entrada no catálogo](../catalog/javascript-typescript.md).

| Code | J | Conf | What it catches |
|---|---|---|---|
| [C2](../catalog/javascript-typescript.md#c2) | J1 | HIGH | corpo do teste sem nenhuma assertion |
| [C5](../catalog/javascript-typescript.md#c5) | J2 | HIGH | assertion sempre verdadeira |
| [C6](../catalog/javascript-typescript.md#c6) | J4 | LOW | assertion fraca: só checa que algo voltou |
| [C7](../catalog/javascript-typescript.md#c7) | J2 | HIGH | autocomparação: os dois lados são idênticos |
| [C8](../catalog/javascript-typescript.md#c8) | J4 | LOW | igualdade exata de float |
| [C9](../catalog/javascript-typescript.md#c9) | J4 | LOW | matcher de exceção amplo demais |
| [CC](../catalog/javascript-typescript.md#cc) | J1 | LOW | assert comentado |
| [C16](../catalog/javascript-typescript.md#c16) | J6 | LOW | resultado depende de tempo, aleatoriedade ou sleep não controlados |
| [C18](../catalog/javascript-typescript.md#c18) | J2 | LOW | comparação de string/repr |
| [C20](../catalog/javascript-typescript.md#c20) | J1 | HIGH | assertion após return/raise/fail incondicional |
| [C21](../catalog/javascript-typescript.md#c21) | J1 | LOW | toda assertion está dentro de condicional; nenhuma roda incondicionalmente |
| [C23](../catalog/javascript-typescript.md#c23) | J6 | LOW | caminho de arquivo absoluto ou relativo ao home hard-coded |
| [C2b](../catalog/javascript-typescript.md#c2b) | J1 | LOW | chama código de produção mas não verifica nada |
| [C37](../catalog/javascript-typescript.md#c37) | J2 | LOW | caso de parametrize duplicado |
| [C44](../catalog/javascript-typescript.md#c44) | J2 | HIGH | tautologia numérica |
| [C48](../catalog/javascript-typescript.md#c48) | J1 | LOW | dark patch: liga uma flag de modo-teste e então asserta |
| [C8b](../catalog/javascript-typescript.md#c8b) | J4 | LOW | igualdade aproximada sem tolerância explícita |
| [C11a](../catalog/javascript-typescript.md#c11a) | J2 | LOW | literal autoconfirmante: o teste atribui e então asserta o mesmo valor |
| [D1](../catalog/javascript-typescript.md#diagnostics) | - | LOW | assertion roulette: vários asserts, nenhum com mensagem |
| [D3](../catalog/javascript-typescript.md#diagnostics) | - | LOW | assert duplicado: a mesma assertion aparece duas vezes |
| [D4](../catalog/javascript-typescript.md#diagnostics) | - | LOW | casos de parametrize sem nome |
| [D6](../catalog/javascript-typescript.md#diagnostics) | - | LOW | print de debug no teste |
| [D7](../catalog/javascript-typescript.md#diagnostics) | - | LOW | teste anônimo: descrição vazia ou ausente |
| [D8](../catalog/javascript-typescript.md#diagnostics) | - | LOW | número mágico numa assertion |
| [JS1](../catalog/javascript-typescript.md#js1) | - | HIGH | teste focado (`it.only`/`fit`) pula o resto da suíte |
| [JS2](../catalog/javascript-typescript.md#js2) | - | HIGH | `expect(x)` sem matcher |
| [JS3](../catalog/javascript-typescript.md#js3) | - | LOW | snapshot é a única assertion |
| [JS4](../catalog/javascript-typescript.md#js4) | - | LOW | teste pulado (`it.skip`/`xit`/`it.todo`) |
| [JS5](../catalog/javascript-typescript.md#js5) | - | LOW | query/evento async sem await (`findBy*`/`waitFor`/user-event) |
| [JS6](../catalog/javascript-typescript.md#js6) | - | HIGH | `describe`/`suite` vazio |
| [JS7](../catalog/javascript-typescript.md#js7) | - | LOW | assertion num callback de `setTimeout`/`then` sem await |
| [JS8](../catalog/javascript-typescript.md#js8) | - | LOW | mocka a unidade sob teste e a asserta diretamente |
| [JS9](../catalog/javascript-typescript.md#js9) | - | HIGH | assertion num ramo literal morto (`if(false)`) |
| [JS11](../catalog/javascript-typescript.md#js11) | - | LOW | `try/catch` engole a assertion |
| [JS13](../catalog/javascript-typescript.md#js13) | - | LOW | query `queryBy*` (retorna null na ausência) como statement solto, nunca asserida; `getBy*`/`findBy*` lançam na ausência e *são* a assertion |
| [JS15](../catalog/javascript-typescript.md#js15) | - | LOW | comparação embrulhada num booleano (`expect(a===b).toBe(true)`) |
| [JS17](../catalog/javascript-typescript.md#js17) | - | LOW | bloco de teste comentado (`// it(...)`) |
| [JS18](../catalog/javascript-typescript.md#js18) | - | LOW | callback `done` em vez de async/await |
| [JS21](../catalog/javascript-typescript.md#js21) | - | HIGH | matcher referenciado mas nunca chamado (`expect(x).toBe` sem `()`) |
| [JS22](../catalog/javascript-typescript.md#js22) | - | HIGH | tabela `it.each`/`test.each` vazia |
| [JS23](../catalog/javascript-typescript.md#js23) | - | HIGH | `expect.assertions(N)` com menos chamadas `expect()` alcançáveis incondicionalmente do que `N` |
| [JS24](../catalog/javascript-typescript.md#js24) | - | LOW | query Cypress `cy.get/find/contains` sem assertion `.should`/`.and`/`.then` |
| [JS25](../catalog/javascript-typescript.md#js25) | - | HIGH | a única assertion fica num callback de iterador de array; roda zero vezes numa coleção vazia |
| [JS26](../catalog/javascript-typescript.md#js26) | - | LOW | fake timers instalados mas nunca avançados; o callback agendado nunca dispara |
| [JS27](../catalog/javascript-typescript.md#js27) | - | LOW | `toHaveBeenCalled*` como único oráculo sobre um dublê criado localmente; verifica a fiação, não o comportamento |
| [JS29](../catalog/javascript-typescript.md#js29) | - | LOW | cadeia `expect(...).resolves`/`.rejects` como statement solto, sem await nem return |
| [JS30](../catalog/javascript-typescript.md#js30) | - | HIGH | assertion literal-vs-literal (`expect(2).toBe(3)`); os dois operandos fixos em parse time |
| [JS31](../catalog/javascript-typescript.md#js31) | - | LOW | `try/catch` engole um throw possível sem assertion sobre a exceção |
| [M2](../catalog/javascript-typescript.md#diagnostics) | - | LOW | método de teste longo |
| [PL7](../catalog/javascript-typescript.md#pl7) | J5 | n/a | sem gate de cobertura |
| [PL8](../catalog/javascript-typescript.md#pl8) | J5 | n/a | execução para cedo |
| [PL10](../catalog/javascript-typescript.md#pl10) | J1 | n/a | `passWithNoTests` |

## Robot Framework (robotframework-falsegreen)

O scanner [robotframework-falsegreen](../scanners/robot.md) emite 30 códigos: os específicos de
`R*` mais os `C*` compartilhados. Cada um liga para sua
[entrada no catálogo](../catalog/robot.md).

| Code | J | Conf | What it catches |
|---|---|---|---|
| [C2](../catalog/robot.md#c2) | J1 | HIGH | corpo do teste sem nenhuma assertion |
| [C3](../catalog/robot.md#c3) | J1 | HIGH | assert dentro de um try cujo except engole o erro |
| [C5](../catalog/robot.md#c5) | J2 | HIGH | assertion sempre verdadeira |
| [C6](../catalog/robot.md#c6) | J4 | LOW | assertion fraca: só checa que algo voltou |
| [C7](../catalog/robot.md#c7) | J2 | HIGH | autocomparação: os dois lados são idênticos |
| [C9](../catalog/robot.md#c9) | J4 | LOW | matcher de exceção amplo demais |
| [CC](../catalog/robot.md#cc) | J1 | LOW | assert comentado |
| [C16](../catalog/robot.md#c16) | J6 | LOW | resultado depende de tempo, aleatoriedade ou sleep não controlados |
| [C20](../catalog/robot.md#c20) | J1 | HIGH | assertion após return/raise/fail incondicional |
| [C21](../catalog/robot.md#c21) | J1 | LOW | toda assertion está dentro de condicional; nenhuma roda incondicionalmente |
| [C23](../catalog/robot.md#c23) | J6 | LOW | caminho de arquivo absoluto ou relativo ao home hard-coded |
| [C2b](../catalog/robot.md#c2b) | J1 | LOW | chama código de produção mas não verifica nada |
| [C31](../catalog/robot.md#c31) | J4 | LOW | resultado da captura descartado |
| [C32](../catalog/robot.md#c32) | J1 | LOW | skip sem motivo |
| [C37](../catalog/robot.md#c37) | J2 | LOW | caso de parametrize duplicado |
| [C44](../catalog/robot.md#c44) | J2 | HIGH | tautologia numérica |
| [C9b](../catalog/robot.md#c9b) | J4 | n/a | RequestsLibrary `expected_status=any` |
| [C11a](../catalog/robot.md#c11a) | J2 | LOW | literal autoconfirmante: o teste atribui e então asserta o mesmo valor |
| [D2](../catalog/robot.md#diagnostics) | J4 | n/a | fluxo de controle a nível de teste |
| [M2](../catalog/robot.md#diagnostics) | - | LOW | método de teste longo |
| [PL9](../catalog/robot.md#pl9) | J1 | n/a | opção de execução skip-on-failure |
| [R1](../catalog/robot.md#r1) | J1 | n/a | verde forçado |
| [R2](../catalog/robot.md#r2) | J1 | n/a | keyword verificadora oca |
| [R3](../catalog/robot.md#r3) | J1 | n/a | test cases num `.resource` |
| [R4](../catalog/robot.md#r4) | J1 | n/a | só `No Operation` |
| [R5](../catalog/robot.md#r5) | J1 | n/a | `[Template]` vazio |
| [R6](../catalog/robot.md#r6) | J4 | n/a | `Should Be True` sobre uma string literal |
| [R7](../catalog/robot.md#r7) | J1 | n/a | keyword de `[Template]` oca |
| [R8](../catalog/robot.md#r8) | J4 | n/a | verificação só no Setup |
| [R8b](../catalog/robot.md#r8b) | J4 | n/a | verificação só no Teardown |

## Semantic codes, skill-only (all languages)

Só a [skill](../scanners/skill.md) LLM detecta estes. Ficam em J2/J3/J4 e precisam de uma leitura
de intenção que um AST não decide. Cada um liga para o
[catálogo semântico](../catalog/semantic.md).

| Code | J | What it catches |
|---|---|---|
| [S1](../catalog/semantic.md#s-series) | J4 | intent mismatch |
| [S2](../catalog/semantic.md#s-series) | J4 | oráculo irrelevante |
| [S3](../catalog/semantic.md#s-series) | J2 | valor esperado plausível-mas-errado |
| [S4](../catalog/semantic.md#s-series) | J4 | oráculo não distingue o correto de um bug provável |
| [S5](../catalog/semantic.md#s-series) | J3 | testa o framework, não o código |
| [S6](../catalog/semantic.md#s-series) | J4 | só happy-path contra um contrato declarado |
| [S7](../catalog/semantic.md#s-series) | J2 | esperado tirado da saída |
| [S8](../catalog/semantic.md#s-series) | J3 | retorno de mock chega na assertion por uma indireção |
| [S9](../catalog/semantic.md#s-series) | J2 | arranjo autorrealizável |
| [S10](../catalog/semantic.md#s-series) | J4 | asserta o log, não o efeito |
| [S11](../catalog/semantic.md#s-series) | J4 | assertion só negativa sobre um filtro de segurança |
| [S12](../catalog/semantic.md#s-series) | J3 | patcheia a lógica central em vez de uma borda externa |
| [S13](../catalog/semantic.md#s-series) | J6 | passa só por estado compartilhado que um irmão montou |
| [S14](../catalog/semantic.md#s-series) | J2 | saída de modelo gravada como oráculo |
| [S15](../catalog/semantic.md#s-series) | J6 | laço de retry/poll feito à mão mascarando flakiness |
| [S16](../catalog/semantic.md#s-series) | J4 | verificação de chamada como único oráculo |
| [S17](../catalog/semantic.md#s-series) | J4 | cegueira de oráculo no caminho de exceção |
| [S18](../catalog/semantic.md#s-series) | J3 | valor de stub impossível pelo contrato |
| [S21](../catalog/semantic.md#s-series) | J2 | assertion de LLM/agente que se autojulga |

## Patterns characteristic of each level { #by-level }

A mesma classe de false-green aparece no nível em que o teste roda. Estes são os clusters
típicos.

### Unit (the bulk of false-greens)

- always-true / tautologia: `C5`, `C7`, `C52`, `JS30`
- sem oráculo / corpo vazio: `C2`, `C2b`, `JS2`
- asserta o próprio dublê / mock: `C13b`, `C55`, `C11a`, `JS8`, `JS27`, `S8` (stub-echo), `S16`
- condicional-only / não roda: `C21`, `JS9`, `C20` (após um terminador), `JS25` (iterador sobre coleção vazia)
- coroutine nunca awaitada: `C56`; comparação solta: `C59`; atributo de Mock não configurado: `C57`
- semânticos: `S5` (testa o framework), `S3` (valor esperado plausível-mas-errado), `S6`

### Integration (crosses the boundary: I/O, DB, HTTP, collaborator)

- oráculo do request desligado: `C9b` (`expected_status=any`)
- captura sem asserir: `C50` (caplog / assertLogs)
- mockar a unidade sob teste em vez da borda: `S12`
- round-trip que só confirma o que você mandou (DB / HTTP liveness vazio)
- semânticos: `S9`, `S10`, `S11`, `S18` (stub com valor impossível pelo contrato)

### E2E (the full stack: browser, flow, hardware)

- sleep como sincronização em vez de `Wait Until`: `C16`
- age e só loga/screenshot sem verificar: `C2b` em browser/login, `R4` (só `No Operation`)
- forced-green: `R1` (`Pass Execution`), `R2` (keyword verificadora oca)
- presença de elemento ou `Should Be True` sobre string como único oráculo: `R6`
- semânticos: `S1` (intent mismatch: o nome promete o que o corpo não verifica), `S2` (oráculo irrelevante)

### Diagnostics (off by default, not false-green)

`D1`, `D3`-`D8`, `M2` são higiene e manutenibilidade (fluxo de controle, teste longo). Ligam com
`--diagnostics`. Não são a tese false-green; ver
[what we do not flag](what-we-do-not-flag.md).
