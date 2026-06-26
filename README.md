# falsegreen-docs

Documentation site for the [falsegreen](https://github.com/vinicq/falsegreen) family:
the deterministic false-green test scanners plus the semantic LLM pass. Built with
[MkDocs Material](https://squidfunk.github.io/mkdocs-material/).

The site carries the unified code catalog (every detection code, its signal, and a
BAD vs CLEAN example), the failure taxonomy (F1-F8) and judgments (J1-J6), and a page
per scanner.

## Local preview

```bash
pip install -r requirements.txt
mkdocs serve
```

Open http://127.0.0.1:8000.

## Build

```bash
mkdocs build --strict
```

The static site lands in `site/`. Deployment to GitHub Pages runs from CI on push to
`main` (`.github/workflows/deploy-docs.yml`).

## The family

- [falsegreen](https://github.com/vinicq/falsegreen) - Python/pytest
- [falsegreen-js](https://github.com/vinicq/falsegreen-js) - JavaScript/TypeScript
- [robotframework-falsegreen](https://github.com/vinicq/robotframework-falsegreen) - Robot Framework
- [falsegreen-skill](https://github.com/vinicq/falsegreen-skill) - semantic LLM pass
