# Credits and references

The catalog is grounded in published research, not invented. This page is the study base: the
literature falsegreen builds on, with DOIs where available. Each repository carries its own
`CREDITS.md` with the code-to-source map; the full denominator cross-walk is on the
[coverage vs the literature](concepts/denominator.md) page.

## Founding work: rotten green and false-green

The false-green axis generalizes the rotten-green-test idea: a test that runs green while its
assertion never executes or never fails.

- Soares, E. A. *A Multimethod Study of Test Smells: Cataloging, Removal, and New Types.* PhD
  thesis, Centro de Informática, Universidade Federal de Pernambuco (UFPE), Recife, 2023. The
  central source: it names the mechanism (Conditional Test Logic produces an unreliable "passed"
  outcome when the assertion is skipped) and a unified catalog of 480 distinct smells from 127
  primary studies.
- Delplanque, J.; Ducasse, S.; Polito, G.; Black, A. P.; Etien, A. *Rotten Green Tests.* ICSE
  2019. The work that names the family.
- Aranega, V. et al. *Rotten green tests in Java, Pharo and Python.* Empirical Software
  Engineering, v. 26, n. 6, p. 130, 2021.
- Martinez, M.; Etien, A.; Ducasse, S.; Fuhrman, C. *RTj: A Java framework for detecting and
  refactoring rotten green test cases.* ICSE Companion, 2020, p. 69-72.
- Kim, D. J. et al. *Studying test annotation maintenance in the wild.* ICSE 2021, p. 62-73.
- Aranda III, M.; Ribeiro, M. *Beyond Green Tests: Removing Smells From Natural Language Tests.*
  SBQS 2025, São José dos Campos.

## Catalog and taxonomy

- van Deursen, A.; Moonen, L.; van den Bergh, A.; Kok, G. *Refactoring Test Code.* XP 2001. The
  founding catalog of 11 test smells plus refactorings.
- Meszaros, G. *xUnit Test Patterns: Refactoring Test Code.* Addison-Wesley, 2007. The source of
  Conditional Test Logic and many structural patterns.
- Palomba, F.; Zaidman, A.; De Lucia, A. *Automatic Test Smell Detection using Information
  Retrieval Techniques.* ICSME 2018, p. 311-322. DOI [10.1109/ICSME.2018.00040](https://doi.org/10.1109/ICSME.2018.00040).
- Aljedaani, W. et al. *Test Smell Detection Tools: A Systematic Mapping Study.* EASE 2021, p.
  170-180. DOI [10.1145/3463274.3463335](https://doi.org/10.1145/3463274.3463335).

## Detection tools and maintainability

- Hauptmann, B.; Eder, S.; Junker, M.; Juergens, E.; Woinke, V. *Generating Refactoring Proposals
  to Remove Clones from Automated System Tests.* ICPC 2015, p. 115-124. DOI
  [10.1109/ICPC.2015.20](https://doi.org/10.1109/ICPC.2015.20).
- Pizzini, A.; Reinehr, S.; Malucelli, A. *Automatic Refactoring Method to Remove Eager Test
  Smell.* SBQS 2022, Curitiba. DOI [10.1145/3571473.3571478](https://doi.org/10.1145/3571473.3571478).
- Fowler, M.; Beck, K. *Refactoring: Improving the Design of Existing Code.* 2nd ed., 2019. The
  vocabulary of smells and behavior-preserving refactoring.

## LLM-based detection

- Peixoto, M. et al. *On the Effectiveness of LLMs for Manual Test Verifications.*
  arXiv:[2409.12405](https://arxiv.org/abs/2409.12405), 2024.
- Melo, R. et al. *Agentic LMs: Hunting Down Test Smells.*
  arXiv:[2504.07277](https://arxiv.org/abs/2504.07277), 2025.

## The denominator

- *Open Catalog of Test Smells.* UFAL / easy-software, [test-smell-catalog.readthedocs.io](https://test-smell-catalog.readthedocs.io/)
  ([repo](https://github.com/easy-software-ufal/catalog-test-smells)). 517 smells, 1621
  references, 166 sources. The named universe precision and recall are measured against; only the
  false-green slice is in scope (see [coverage vs the literature](concepts/denominator.md)).

## Published baselines (comparison context)

Results from the published literature, used as comparison points. Our own evaluation against the
denominator lives in the research hub and is released when published.

| Source | Task | Metric | Value |
|---|---|---|---|
| Soares 2023 (NLP) | detect 7 manual smells | P / R / F1 | 0.92 / 0.95 / 93.53% |
| Aranda 2025 (NLP) | detect + remove 7 manual smells | F1 | 83.70% |
| Palomba 2018 (TASTE) | detect Eager Test | F1 | 76% (structural baseline 47%) |
| Palomba 2018 (TASTE) | detect General Fixture | F1 | 67% (structural baseline 23%) |
| Melo 2025 (4 agents) | detect 5 smells (Java) | pass@5 | 96% |
| Melo 2025 (Phi-4-14B) | refactor 5 smells | pass@5 | 75.3% |
| Pizzini 2022 | remove Eager Test automatically | removal rate | 99.4% |

A recurring finding across these: detecting Conditional Test Logic is easy (~96%), but removing it
correctly - without trading a false-green for a false-red - is the hard part (~10% in Melo 2025).
That gap is exactly what the [oracle hierarchy](concepts/oracle.md) and the
[AI-fix gate](scanners/skill.md) address.

## The oracle problem

The expected-value problem has a canonical reference, and the standards body treats it as
first-class:

- Barr, E. T.; Harman, M.; McMinn, P.; Shahbaz, M.; Yoo, S. *The Oracle Problem in Software
  Testing: A Survey.* IEEE TSE, 2015. DOI [10.1109/TSE.2014.2372785](https://doi.org/10.1109/TSE.2014.2372785).
  The ISTQB CT-AI syllabus relies on it.
- Segura, S. et al. *A Survey on Metamorphic Testing.* IEEE TSE, 2016; and *Metamorphic Testing:
  Testing the Untestable.* IEEE Software, 2020. Oracle-problem solutions.
- Wiegers, K.; Beatty, J. *Software Requirements.* 3rd ed., 2013. A verifiable requirement is one
  you can build an oracle for; a green test whose traced spec changed is a "suspect link"; the
  "self-fulfilling prophecy" of testing against the code is the F7 failure in requirements terms.

The [oracle hierarchy](concepts/oracle.md) is the practical answer: the expected value must come
from a source independent of the code.

## Vocabulary

The taxonomy aligns with the standard testing vocabulary (the ISTQB glossary: test oracle, defect
vs failure, test level) while keeping the product term **false-green** for the specific failure
mode: a test that passes while protecting nothing.

A precise distinction, easy to get wrong: in ISTQB terms a **false-positive** is a spurious
failure report (a test that fails without a real defect). **false-green is the opposite** - a test
that *passes* without protecting anything, which enables a **false-negative** (an escaped defect).
The chain that backs the product is: a defect produces no failure on this run, the test stays
green, the defect ships. Coverage percentage hides this; the Defect Detection Percentage (defects
found by testing over total) does not, which is why the evaluation reports precision and recall
against the false-green slice rather than coverage.

## How to cite

If you use falsegreen in academic work, cite the relevant product repository and the founding
rotten-green-test literature above. A canonical citation entry is added here once the study is
published. See the [research foundation](research.md) for the methodology.
