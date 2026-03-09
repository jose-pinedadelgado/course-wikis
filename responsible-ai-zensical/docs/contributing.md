# Contributing

This wiki is maintained as part of the Chalq project at CSULB.

## How to Contribute

1. Fork the repository
2. Create a feature branch: `git checkout -b my-contribution`
3. Add or edit content in `docs/modules/`
4. Submit a pull request

## Style Guide

- Use **admonitions** for tips, warnings, and examples
- Include **Python code** with syntax highlighting
- Use **MathJax** for formulas: `$$..$$` for display, `$...$` for inline
- Use **Mermaid** diagrams to illustrate concepts
- Keep language accessible — explain jargon before using it

## Building Locally

```bash
uv sync
.venv/Scripts/python -m zensical serve
```

Then open `http://localhost:8000` in your browser.
