# falsegreen (Python)

The deterministic Python/pytest scanner. A zero-dependency AST pass that validates each test
against the false-positive codes a parser can prove. HIGH findings block the commit, LOW ones
warn, and a diagnostic/coupling group is opt-in.

- Repository: [github.com/vinicq/falsegreen](https://github.com/vinicq/falsegreen)
- Catalog: [Python codes](../catalog/python.md)

## Install

```bash
pip install falsegreen
```

## Use

```bash
falsegreen path/to/tests        # scan
falsegreen --staged             # only staged files (pre-commit)
falsegreen --json               # machine-readable report
falsegreen --diagnostics        # include the opt-in F8 group
falsegreen --config-audit       # read pytest/coverage config for project-level false-green
```

HIGH findings exit non-zero, so the tool drops into CI and pre-commit unchanged. The report
numbers each finding with its code, judgment, pyramid level, location, evidence, and a fix hint.

## Scope

The scanner owns the structural slice (F1-F6 a parser can prove). The semantic slice lives in
[falsegreen-skill](skill.md); runtime is out of band. See
[scope and honesty](../concepts/scope.md) and [what we don't flag](../concepts/denominator.md).
