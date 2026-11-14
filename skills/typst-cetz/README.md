# typst-cetz-skill

An LLM skill for [CeTZ](https://cetz-package.github.io/docs/) diagrams in [Typst](https://github.com/typst/typst) -- TikZ-inspired drawing without the LaTeX overhead.

## What does it do?

When you ask an LLM to create diagrams in Typst, it often invents non-existent API or falls back on TikZ syntax. This skill gives the model a structured reference for CeTZ's coordinate systems, drawing primitives, styling, anchors, marks, trees, and plotting so it produces working diagrams on the first try.

## Installation

### Claude Code (one command)

```bash
git clone https://github.com/statzhero/typst-cetz-skill.git ~/.claude/skills/typst-cetz
```

The skill is available immediately as `/typst-cetz` in any Claude Code session.

### Codex

```bash
git clone https://github.com/statzhero/typst-cetz-skill.git ~/.agents/skills/typst-cetz
```

### Other LLMs

Paste the contents of `SKILL.md` into your system prompt or attach it as context.

## Related skills

- [typst-skill](https://github.com/statzhero/typst-skill) -- Core Typst syntax, styling, math, tables, figures
- [typst-touying-skill](https://github.com/statzhero/typst-touying-skill) -- Slide presentations with Touying

## License

CC-BY-4.0
