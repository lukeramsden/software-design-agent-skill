# Software Design Agent Skill

A practical software-design review skill based on the concepts and diagnostic vocabulary in John Ousterhout's *A Philosophy of Software Design*.

It helps coding agents identify complexity, information leakage, shallow modules, temporal decomposition, vague abstractions, and the other named design red flags. It is intended for code review, architecture discussions, refactoring, and technical writing.

## Install

Install with the Skills CLI:

```bash
npx skills add lukeramsden/software-design-agent-skill@software-design
```

Install globally for all supported agents:

```bash
npx skills add lukeramsden/software-design-agent-skill@software-design -g -a '*' -y
```

List the skills available in this repository without installing:

```bash
npx skills add lukeramsden/software-design-agent-skill --list
```

## What the skill provides

- Ousterhout's 15 software-design principles
- The 14 canonical red flags, with their canonical names
- Diagnostic vocabulary for complexity and obscurity
- Guidance on deep modules, information hiding, layering, errors, comments, naming, consistency, and performance
- A practical code-review checklist

## Repository layout

```text
skills/
└── software-design/
    └── SKILL.md
```

## Attribution

This is an independent practitioner's distillation inspired by John Ousterhout's *A Philosophy of Software Design* (2018). It is not affiliated with or endorsed by John Ousterhout or the book's publisher. Read the original book for the complete argument, examples, and context.

## License

The original material in this repository is available under the MIT License. The names and ideas attributed to John Ousterhout remain the work of their respective author and rights holders. See [NOTICE](NOTICE).
