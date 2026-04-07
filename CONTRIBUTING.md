# Contributing to FAOS Skills Marketplace

Thank you for your interest in contributing! This guide covers how to add skills, improve existing
ones, and submit changes.

## Ways to Contribute

### 1. Submit a New Skill

Skills are standalone instruction files that teach AI assistants how to perform specific tasks.

**Requirements:**
- One skill per file, self-contained (no external dependencies)
- Clear, actionable instructions — tested on at least one platform
- Appropriate category placement (engineering, product, data-ai, growth, leadership, creative)
- YAML frontmatter with `name`, `description`, and `tags`

**Process:**
1. Fork this repository
2. Create your skill in the appropriate `skills/cowork/{category}/` directory
3. Follow the naming convention: `{skill-name}/SKILL.md`
4. Submit a pull request with a description of what the skill does

We'll handle cross-platform exports (Codex, Gemini, Copilot, Perplexity) from the Cowork source.

### 2. Improve an Existing Skill

Found a skill that could be clearer, more accurate, or more effective? Open a PR with your
improvements. Please include:
- Which skill you're improving
- What you changed and why
- Which platform(s) you tested on

### 3. Report Issues

- **Skill doesn't work as expected** — open an issue with the skill name, platform, and what happened
- **Missing skill for a common workflow** — open a feature request describing the use case
- **Platform compatibility issue** — tell us which platform and what broke

## Skill Format

### Cowork (SKILL.md)

```yaml
---
name: my-skill-name
description: One-line description of what this skill does
tags: [category, subcategory]
---

# Skill Title

Instructions for the AI assistant...
```

### Agent Plugin

Agent plugins live in `plugins/faos-{role}/` and follow the YAML agent format:

```
plugins/faos-{role}/
├── agent.yaml       # Agent persona, menu, vocabulary
└── README.md        # Usage instructions
```

## Pull Request Guidelines

1. **One skill per PR** (or a coherent set of related skills)
2. **Descriptive title:** `feat: add {skill-name} skill for {use-case}`
3. **Test your skill** on at least one platform before submitting
4. **No proprietary content** — skills must be original or properly attributed
5. **Apache 2.0 compatible** — your contribution will be licensed under Apache 2.0

## Development Setup

```bash
# Clone the repo
git clone https://github.com/frank-luongt/faos-skills-marketplace.git
cd faos-skills-marketplace

# No build step required — skills are plain text files
# Just edit and test directly with your AI platform
```

## Questions?

- Open a [GitHub Discussion](https://github.com/frank-luongt/faos-skills-marketplace/discussions)

## License

By contributing, you agree that your contributions will be licensed under the Apache License 2.0.
