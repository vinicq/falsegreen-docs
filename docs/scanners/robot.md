# robotframework-falsegreen (Robot Framework)

The deterministic Robot Framework scanner. A static scan over the official parser
(`robot.api.get_model`), no execution. It recognizes the verification vocabulary across the Robot
library ecosystem, so a real check is not mistaken for "no oracle".

- Repository: [github.com/vinicq/robotframework-falsegreen](https://github.com/vinicq/robotframework-falsegreen)
- Catalog: [Robot Framework codes](../catalog/robot.md)
- CLI command: `rffalsegreen`

## Install

```bash
pip install robotframework-falsegreen
```

## Use

```bash
rffalsegreen path/to/suite              # scan
rffalsegreen --format json|sarif|junit  # output shape
rffalsegreen --config-audit             # robot.toml / invocation for project-level checks
```

## Scope

Static scan: it owns what the keyword structure proves. It does not run the suite, so runtime-only
smells (Test Run War, cross-suite order dependence) are out of band, and the semantic slice lives
in [falsegreen-skill](skill.md). Codes share an id with the siblings where the concept matches;
`R*` codes are Robot-specific. See [scope and honesty](../concepts/scope.md).
