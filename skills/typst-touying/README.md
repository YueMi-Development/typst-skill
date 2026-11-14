# typst-touying-skill

An LLM skill for [Touying](https://touying-typ.github.io/touying/) slide presentations in [Typst](https://github.com/typst/typst) -- fast, themeable slides without Beamer.

## What does it do?

When you ask an LLM to create Typst slides, it often guesses at the API or mixes up theme syntax. This skill gives the model a structured reference for Touying's themes (simple, university, metropolis, aqua, dewdrop, stargazer), slide animations with `#pause` and `#meanwhile`, multi-file presentations, and CeTZ/Fletcher diagram integration.

## Installation

### Claude Code (one command)

```bash
git clone https://github.com/statzhero/typst-touying-skill.git ~/.claude/skills/typst-touying
```

The skill is available immediately as `/typst-touying` in any Claude Code session.

### Codex

```bash
git clone https://github.com/statzhero/typst-touying-skill.git ~/.agents/skills/typst-touying
```

### Other LLMs

Paste the contents of `SKILL.md` into your system prompt or attach it as context.

## Related skills

- [typst-skill](https://github.com/statzhero/typst-skill) -- Core Typst syntax, styling, math, tables, figures
- [typst-cetz-skill](https://github.com/statzhero/typst-cetz-skill) -- CeTZ diagrams (TikZ-style drawing)

## License

CC-BY-4.0
