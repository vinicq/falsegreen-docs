# Catálogo semântico (LLM)

Os padrões que só a [skill](../scanners/skill.md) consegue pegar. Eles exigem ler o teste contra
sua intenção, a spec e o código de produção: nenhum parser ou linter os vê. Esta é a camada F7,
e a razão de a skill existir em cima dos scanners estáticos.

A confiança nestes é confirmada pelo operador: trate como BAIXO ou ALTO conforme o quão clara a contradição
é. Nunca bloqueie automaticamente sem mostrar o raciocínio, e nunca reporte um achado de valor-errado sem
citar um [oráculo](../concepts/oracle.md) independente.

## O catálogo de casos

Estes cinco casos são os false-greens semânticos canônicos, detectados ao ler o teste.

### Caso 10 - faz mock da unidade sob teste
`J3` · ALTO

O teste faz patch ou mock da função que deveria testar, depois afirma sobre o valor de retorno do
mock. Ele testa a configuração do mock, não o código.

=== "RUIM"
    ```python
    @patch('mymodule.add')
    def test_add(mock_add):
        mock_add.return_value = 5
        assert add(2, 3) == 5          # afirma o valor do mock
    ```
=== "LIMPO"
    ```python
    @patch('mymodule.db.fetch')        # mock na borda, não na unidade
    def test_get_user(mock_fetch):
        mock_fetch.return_value = {'id': 1, 'name': 'Alice'}
        user = get_user(1)
        assert user.name == 'Alice'
    ```

### Caso 11 - afirma o valor alimentado ao mock
`J2` / `J3` · ALTO

O teste faz stub de uma dependência para retornar X, depois afirma que o resultado é igual a X. O resultado passa
por nenhuma lógica de produção - é um eco.

=== "RUIM"
    ```python
    def test_price(mock_product):
        mock_product.price = 100
        assert get_price(mock_product) == 100   # ecoa o stub
    ```
=== "LIMPO"
    ```python
    def test_price_with_tax(mock_product):
        mock_product.price = 100
        assert get_price_with_tax(mock_product) == 110   # lógica real
    ```

### Caso 12 - reimplementa a fórmula de produção
`J2` · ALTO

O valor esperado é calculado com a mesma fórmula do código. Ambos concordam na mesma resposta
errada.

=== "RUIM"
    ```python
    def test_total():
        expected = price + price * tax_rate   # reimplementa o SUT
        assert calculate_total(price, tax_rate) == expected
    ```
=== "LIMPO"
    ```python
    def test_total():
        assert calculate_total(100, 0.1) == 110.0   # da spec
    ```

### Caso 15 - passa só se outro teste rodou antes
`J6` · ALTO

O teste lê estado mutável compartilhado que um irmão configurou. Ele passa numa ordem específica e falha quando
rodado sozinho.

### Caso 18 - o valor esperado contradiz o que o código deveria fazer
`J2` · ALTO

O teste afirma um valor que contradiz a spec, congelando um bug como correto. **Exige um
oráculo independente citado antes de reportar.**

## A série S (só IA)

Padrões que nenhum AST ou linter vê. Cada um mapeia para um julgamento.

| Código | J | O que detecta |
|---|---|---|
| **S1** | J4 | descompasso de intenção: o nome diz verificar X, a asserção verifica Y ou uma propriedade trivial |
| **S2** | J4 | oráculo irrelevante: afirma uma propriedade não relacionada ao comportamento sob teste |
| **S3** | J2 | valor esperado plausível-mas-errado (off-by-one, arredondamento errado); mais profundo que o caso 18 |
| **S4** | J4 | o oráculo não distingue correto de um bug provável (`len(result) == 3` quando o bug também dá três) |
| **S5** | J3 | testa o framework, não o código (um dict guarda uma chave, o ORM retorna o que foi salvo) |
| **S6** | J4 | só caminho feliz contra um contrato declarado que promete tratamento de erro |
| **S7** | J2 | valor esperado tirado da saída (um dict colado, uma resposta capturada) |
| **S8** | J3 | o retorno do mock alcança a asserção por uma indireção |
| **S9** | J2 | arranjo autorrealizável: arranja o exato estado que depois afirma |
| **S10** | J4 | afirma o log, não o efeito que a mensagem descreve |
| **S11** | J4 | asserção só-negativa em um filtro de segurança (`secret not in output`) sem positiva pareada |
| **S12** | J3 | faz patch na lógica central em vez de uma borda externa (mais profundo que o caso 10) |
| **S13** | J6 | passa só via estado compartilhado que um irmão configurou, entre arquivos que o AST não prova |

## Parecidos: NÃO sinalizar

- Um teste de unidade deliberadamente estreito cujo escopo a spec confirma (S6 precisa de um contrato
  mais amplo declarado).
- Uma constante que a spec de fato endossa (não S3).
- Um teste de sanitizador que já pareia a verificação negativa com uma positiva (não S11).
- Um mock em uma borda externa genuína: DB, rede, relógio (não S12).
- Um teste cujo estado compartilhado é reiniciado por um teardown autouse / `beforeEach` (não S13).
