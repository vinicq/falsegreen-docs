# What we do not flag

Este é o espaço negativo do catálogo: o que foi deliberadamente deixado de fora, e a razão. É a
resposta de escopo e ameaças à validade, e a réplica canônica pro "por que X não é sinalizado".
Cada entrada nomeia a razão real. Para os padrões que *são* cobertos, ver
[patterns by language and test level](by-test-level.md).

A assimetria é intencional: o catálogo prefere precisão (poucos falsos positivos) a recall. Um
código rejeitado aqui não é esquecido; é uma decisão registrada. Reabrir um exige nova
adjudicação, não inclusão unilateral.

## The six reason categories { #reasons }

### 1. Belongs to the semantic skill, not static AST { #belongs-to-skill }

Um AST decide estrutura, não intenção. Estes só viram S-codes ou um julgamento J2-J4, nunca um
`C`/`JS`/`R`:

- **Mystery Guest completo** (além do IP/URL fixo do `C23`): nomear uma dependência externa
  oculta exige semântica. A fatia estática segura para no `C23`; o resto é da skill.
- **Eager Test** (um teste exercitando muitas unidades): é design, não um false-green que um AST
  decide.
- **Assert-the-mock profundo** (o valor passa por uma indireção não-trivial antes de bater no
  mock): o caso estático raso é `C13b`/`JS27`/`C11a`; o profundo é `S8`/`S16` na skill.
- **Literal mágico como oráculo circular**: vira o eixo semântico, não um `C`-code.

### 2. Precision-first: the false-positive ceiling is too high { #fp-ceiling }

Um código que dispararia sobre código legítimo é pior que a ausência (um falso positivo custa
mais que um miss). Rejeitados:

- **`C8b` no Robot** (tolerância numérica aproximada): o texto não-tipado do Robot não dá sinal
  estático para dimensionar a tolerância. Skipped no Robot.
- **Global / shared state estático**: detectar acoplamento de estado entre testes por AST produz
  falsos positivos demais. Fica como `S13` (skill) quando há evidência.
- **Resource Optimism além do `C23`**: assumir que um recurso existe. A fatia segura é o `C23`;
  ir além super-dispara.
- **`C2b` delegate-helper (single-file ceiling)**: quando a verificação mora num helper de outro
  arquivo que o scanner single-file não resolve, o `C2b` fica LOW e com ressalva, não promovido a
  HIGH.

### 3. Not a false-green: another smell axis (maintainability / duplication) { #not-a-false-green }

A tese é false-green: passar verde sem proteger nada. Smells de manutenção ficam OFF por padrão
(diagnósticos):

- **Clone / duplicação de teste**: qualidade, não false-green. Não é código.
- **Teste longo demais / fluxo de controle no teste**: `M2` e os `D*` são diagnósticos, OFF por
  padrão, território Robocop / tsDetect-maintainability. Ligam com `--diagnostics`.

### 4. Already covered by an existing code (a new code would double-report) { #already-covered }

- **Corpo inerte de `pytest.raises`**: sobrepõe o `C51` (raises vazio) ou exige a semântica do
  `S17`; um código novo criaria double-report ou uma máquina de falsos positivos. Rejeitado.
- **Smells de PyNose / pytest-smell / TEMPY (Py) e SNUTS.js / useless-test-detector / AromaDr (JS)
  que mapeiam em `C`/`JS` existentes**: Empty -> `C2`/`C2b`, Conditional -> `C21`/`JS9`,
  Exception -> `C3`/`C17`/`C27`/`JS11`/`JS31`, Sleepy -> `C16`, Redundant -> `C5`/`C7`/`JS30`,
  Skip -> `C25`/`C32`/`C43`/`JS4`/`JS17`, Suboptimal Assert -> `C34`, over-mock -> `JS8`. Não
  viram código novo.

### 5. The premise is factually wrong (not a false-green as stated) { #wrong-premise }

- **Robot `Run Keyword And Continue On Failure` como "swallow"**: na verdade *falha* o teste no
  fim, não engole. O detector seria inútil ou incorreto. Rejeitado.

### 6. Deferred: real, but needs bounding / field validation before it enters { #deferred }

- **Valor capturado nunca usado (caso amplo)**: só o canto AST-decidível entrou, como `C57`
  (comparação contra atributo de Mock não configurado) e o `C31` do Robot (captura nunca
  referenciada). O resto (uso só em Log/Evaluate/teardown) precisa de bounding de falsos
  positivos, uma segunda passada.
- Qualquer candidato novo de detector entra como PROVISIONAL e só é "estável" após field
  validation num repo real. `C56`/`C57`/`C59` passaram por isso (zero over-fire em 483 repos).

## A note on method

Um código rejeitado é uma decisão, não um esquecimento. O catálogo se apoia na precisão: o verde
que ele te dá significa algo porque a linha de escopo é desenhada de propósito e mantida. Para o
quadro completo medido contra a literatura publicada de test-smell, ver
[scope and honesty](scope.md) e [coverage versus the literature](denominator.md).
