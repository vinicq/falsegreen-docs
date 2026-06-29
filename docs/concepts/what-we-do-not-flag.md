# What we do not flag

This is the negative space of the catalog: what was deliberately left out, and the reason. It is
the scope and threats-to-validity answer, and the canonical reply to "why is X not flagged".
Each entry names the real reason. For the patterns that *are* covered, see
[patterns by language and test level](by-test-level.md).

The asymmetry is intentional: the catalog prefers precision (few false positives) over recall.
A code rejected here is not forgotten; it is a recorded decision. Reopening one needs a fresh
adjudication, not a unilateral add.

## The six reason categories { #reasons }

### 1. Belongs to the semantic skill, not static AST { #belongs-to-skill }

An AST decides structure, not intent. These only become S-codes or a J2-J4 judgment, never a
`C`/`JS`/`R`:

- **Full Mystery Guest** (beyond the fixed IP/URL of `C23`): naming a hidden external dependency
  needs semantics. The safe static slice stops at `C23`; the rest is the skill's.
- **Eager Test** (one test exercising many units): a design concern, not a false-green an AST can
  decide.
- **Deep assert-the-mock** (the value passes through a non-trivial indirection before it hits the
  mock): the shallow static case is `C13b`/`JS27`/`C11a`; the deep one is `S8`/`S16` in the
  skill.
- **Magic literal as a circular oracle**: this is the semantic axis, not a `C`-code.

### 2. Precision-first: the false-positive ceiling is too high { #fp-ceiling }

A code that would fire on legitimate code is worse than its absence (a false positive costs more
than a miss). Rejected:

- **`C8b` in Robot** (approximate numeric tolerance): Robot's untyped text gives no static signal
  to size the tolerance. Skipped in Robot.
- **Static global / shared state**: detecting state coupling between tests by AST produces too
  many false positives. It stays as `S13` (skill) when there is evidence.
- **Resource Optimism beyond `C23`**: assuming a resource exists. The safe slice is `C23`; going
  further over-fires.
- **`C2b` delegate-helper (single-file ceiling)**: when the verification lives in a helper in
  another file that the single-file scanner cannot resolve, `C2b` stays LOW and caveated, not
  promoted to HIGH.

### 3. Not a false-green: another smell axis (maintainability / duplication) { #not-a-false-green }

The thesis is false-green: passing green without protecting anything. Maintenance smells stay off
by default (diagnostics):

- **Test clone / duplication**: quality, not false-green. Not a code.
- **Test too long / control flow in the test**: `M2` and the `D*` codes are diagnostics, off by
  default, the Robocop / tsDetect-maintainability territory. They turn on with `--diagnostics`.

### 4. Already covered by an existing code (a new code would double-report) { #already-covered }

- **An inert `pytest.raises` body**: overlaps `C51` (empty raises) or needs the semantics of
  `S17`; a new code would create a double-report or a false-positive machine. Rejected.
- **PyNose / pytest-smell / TEMPY (Py) and SNUTS.js / useless-test-detector / AromaDr (JS) smells
  that map onto existing `C`/`JS`**: Empty -> `C2`/`C2b`, Conditional -> `C21`/`JS9`, Exception ->
  `C3`/`C17`/`C27`/`JS11`/`JS31`, Sleepy -> `C16`, Redundant -> `C5`/`C7`/`JS30`, Skip ->
  `C25`/`C32`/`C43`/`JS4`/`JS17`, Suboptimal Assert -> `C34`, over-mock -> `JS8`. These do not
  become new codes.

### 5. The premise is factually wrong (not a false-green as stated) { #wrong-premise }

- **Robot `Run Keyword And Continue On Failure` as a "swallow"**: it actually *fails* the test at
  the end, it does not swallow. The detector would be useless or incorrect. Rejected.

### 6. Deferred: real, but needs bounding / field validation before it enters { #deferred }

- **A captured value never used (the broad case)**: only the AST-decidable corner entered, as
  `C57` (comparison against an unconfigured Mock attribute) and the Robot `C31` (capture never
  referenced). The rest (use only in Log/Evaluate/teardown) needs false-positive bounding, a
  second pass.
- Any new detector candidate enters as PROVISIONAL and is only "stable" after field validation on
  a real repo. `C56`/`C57`/`C59` went through this (zero over-fire across 483 repos).

## A note on method

A rejected code is a decision, not an oversight. The catalog leans on precision: the green it
gives you means something because the line of scope is drawn on purpose and held. For the full
picture measured against the published test-smell literature, see
[scope and honesty](scope.md) and [coverage versus the literature](denominator.md).
