# Credits and references

The catalog is grounded in published research, not invented. This page names the base; each
repository carries its own `CREDITS.md` with the full code-to-source map.

## The founding work

- **Rotten green tests** - the work that names this whole family: a test that runs green while
  its assertions never execute or never fail. Delplanque, Ducasse, Polito, Black, Etien (ICSE
  2019), and Soares' follow-up (2023). The false-green axis is a generalization of this idea
  across languages and mechanisms.
- **The test-smell refactoring catalog** - van Deursen, Moonen, van den Bergh, Kok (XP 2001), and
  the `xUnit Test Patterns` catalog (Meszaros, 2007), which name many of the structural patterns.

## The denominator

The family measures precision and recall against the
[Open Catalog of Test Smells](https://test-smell-catalog.readthedocs.io/) (UFAL / easy-software):
517 documented smells across 1621 references and 166 sources. Only the false-green slice is in
scope; see [coverage vs the literature](concepts/denominator.md) for what stays out and why. The
full cross-walk lives in the private research hub; only the denominator and the boundary are
public.

## Vocabulary

The taxonomy aligns with the standard testing vocabulary (ISTQB glossary: test oracle, defect vs
failure, test level) while keeping the product term **false-green** for the specific failure mode:
a test that passes without protecting anything. This is narrower than a general "test smell" and
distinct from a false-fail (a test that breaks without a real defect), which is the opposite axis
and out of scope.

## Per-language evidence

- **JavaScript / TypeScript** - empirical false-green studies on real corpora (Jorge, UFCG 2023;
  Silva, PUC Minas 2022) inform the `JS*` codes and the high-prevalence traps.
- **Robot Framework** - the official `HowToWriteGoodTestCases` guide and the RF style guide inform
  the verification vocabulary and the `R*` codes.

Each finding cites the judgment ([J1-J6](concepts/judgments.md)) and family
([F1-F8](concepts/taxonomy.md)) it belongs to, so the academic claim behind every code is
traceable.
