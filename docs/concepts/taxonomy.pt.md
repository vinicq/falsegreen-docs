# Taxonomia de falhas (F1-F8)

Cada código do catálogo mapeia para um modo de falha. A taxonomia é o eixo conceitual: ela
responde *como* um teste fica verde sem proteger nada, independente da linguagem. A política
do produto (bloquear, alertar ou diagnóstico) é um eixo separado, decidido por código.

| Família | Modo de falha | Eixo de risco | Camada que resolve |
|---|---|---|---|
| **F1** | Não verifica nada (sem oráculo) | efetividade | estática, por arquivo |
| **F2** | A verificação existe mas nunca roda | execução | estática, por arquivo |
| **F3** | A verificação é trivial (sempre passa) | efetividade | estática, por arquivo |
| **F4** | Verifica a coisa errada | efetividade (oráculo) | estática (parcial) + skill |
| **F5** | O teste some da contagem (skip / não coletado) | execução | estática + camada de projeto |
| **F6** | Passa ou falha por sorte (não-determinismo) | não-determinismo | estática (proxy) + runtime |
| **F7** | Oráculo circular ou semântico | semântico | skill + mutation testing |
| **F8** | Higiene / legibilidade (não é false-green) | estrutura | diagnóstico opcional / linter |

## Como ler

**F1-F6 são false-green.** O teste reporta sucesso enquanto o código que ele cobre pode estar
quebrado. Esses bloqueiam ou alertam, conforme a confiança.

**F7 é semântico.** O oráculo é circular (o teste re-deriva o valor esperado a partir do
código, ou faz mock da unidade que diz testar). Nenhum parser prova intenção. A
[skill](../scanners/skill.md) lê isso; um gate ao vivo com mutation testing confirma.

**F8 não é false-green.** O teste ainda protege; só é difícil de ler ou manter
(roleta de assertion, corpo longo demais, número mágico). Esses são diagnósticos, desligados por
padrão, e linters dedicados (ruff, ESLint, Robocop) também cobrem. Onde um linter cobre, os
scanners cedem o lugar.

## Por que um eixo separado dos grupos de produto

Um scanner organiza a saída em três grupos de CLI: **false-positive** (bloqueia ou alerta),
**diagnóstico** (opcional) e **acoplamento**. Isso é a *política*. F1-F8 é o *mapa*. Os dois
não colidem: F1-F6 caem no grupo false-positive, F7 vai para a skill e o mutation testing,
F8 é o grupo de diagnóstico.

A taxonomia também nomeia o denominador da medição. Quando o catálogo é cruzado com a
literatura publicada de test-smells, quase todo smell externo mapeia para um modo F1-F8, o que
mantém as afirmações acadêmicas honestas: as ferramentas medem contra um conjunto nomeado de
modos de falha, não uma lista aberta.

## As duas camadas que um parser não alcança

Um arquivo limpo não é uma suíte limpa. Dois modos de falha sobrevivem a um scan por arquivo:

- **F5 no nível de projeto.** O arquivo faz a assertion certa, mas o runner está configurado
  para deixar passar uma execução vazia ou parcial: `--passWithNoTests`, um gate de cobertura
  que nunca é aplicado, um `filterwarnings` que nunca vira erro. O modo `--config-audit` lê a
  configuração do projeto e da CI para pegar isso.
- **F7 no nível semântico.** A assertion roda e parece específica, mas o valor esperado vem do
  próprio código, ou o mock faz as vezes da unidade sob teste. Isso exige ler o teste contra a
  intenção dele, que é a skill, e provar com mutation, que é runtime.

Veja [julgamentos (J1-J6)](judgments.md) para as perguntas por teste que decidem a qual família
um achado pertence.
