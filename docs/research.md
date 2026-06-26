# Research foundation

falsegreen is a research project as much as a tool. It has a dual purpose: **academic** - a
defensible taxonomy, a named denominator, threats to validity - and **industrial** - low false
positives, real patterns, something that runs in CI. Every code in the catalog traces back to a
failure mode and a judgment, so the claim behind it is checkable, not folklore.

## The methodology (our base)

The whole approach rests on four pillars, each with its own page:

- **[Failure taxonomy F1-F8](concepts/taxonomy.md)** - the conceptual axis: *how* a test passes
  green without protecting anything, independent of language.
- **[Judgments J1-J6](concepts/judgments.md)** - six questions asked of a single test; a finding
  names the exact guarantee that fails, not a vague smell.
- **[The oracle hierarchy](concepts/oracle.md)** - the expected value must come from a source
  independent of the code; promoting the code itself to oracle is how a bug freezes as "correct".
- **[The AI-fix gate (F7)](scanners/skill.md)** - a bidirectional mutation gate: a strengthened
  test must pass on clean code and fail on the reintroduced bug, or it is rejected.

## The denominator and threats to validity

Precision and recall are reported against a named universe, not an open-ended list. The family
measures against the [Open Catalog of Test Smells](https://test-smell-catalog.readthedocs.io/)
(517 documented smells, 1621 references, 166 sources), and only the false-green slice is in scope.
What stays out and why is on the [coverage vs the literature](concepts/denominator.md) page - that
page *is* the threats-to-validity statement in public form.

## Baselines from the literature

For comparison context, the published detectors and studies in the adjacent space:

| Tool / study | Precision | Recall | F1 | Scope |
|---|---|---|---|---|
| xNose (Paul, 2024) | 96.97% | 96.03% | - | C#, 16 smells |
| srcML (Lopes, 2023) | 87.25% | 100% | - | C++ and Java, 7 smells |
| JNose (Goes, 2024) | 85-100% | 90-100% | - | Java, 6 smells |
| LLM CoT + one-shot (Santana, 2025) | - | - | 0.732 Py / 0.763 Java | Python and Java |

Our own evaluation against this denominator lives in the research hub; the numbers are released
when they are published, not before.

## The study

The product code and this documentation are public. The dataset, the per-smell adjudication, and
unpublished results live in a **private research hub**, so no unpublished number or evidence
appears in a public repository. Results and any paper are linked here when published.

Public study materials:

- [falsegreen](https://github.com/vinicq/falsegreen) (Python), [falsegreen-js](https://github.com/vinicq/falsegreen-js)
  (JS/TS), [robotframework-falsegreen](https://github.com/vinicq/robotframework-falsegreen)
  (Robot), [falsegreen-skill](https://github.com/vinicq/falsegreen-skill) (semantic).
- The founding work and full reference list: [credits and references](credits.md).
- The literature denominator: [Open Catalog of Test Smells](https://test-smell-catalog.readthedocs.io/).

## How to cite

If you use falsegreen in academic work, cite the relevant product repository and the founding
rotten-green-test literature listed in [credits](credits.md). A canonical citation entry is added
here once the study is published.
