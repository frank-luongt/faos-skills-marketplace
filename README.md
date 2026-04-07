# FAOS Skills Marketplace

> **520+ AI-powered skills and 31 agent plugins** for Claude Cowork, OpenAI Codex, Gemini CLI, GitHub Copilot, and Perplexity Computer.

Built by the [FAOS Framework](https://faosx.ai) — the enterprise agentic operating system.

## Quick Install

### Claude Cowork

```bash
# Install a single skill
cp skills/cowork/engineering/dev-story/SKILL.md .claude/skills/

# Install an agent plugin (e.g., CTO persona)
cp -r plugins/faos-cto/ ~/.claude/plugins/faos-cto/
```

### OpenAI Codex

```bash
# Copy skills to your Codex project
cp -r skills/codex/ .agents/skills/

# The agents/openai.yaml provides UI metadata automatically
```

### Gemini CLI

```bash
# Copy all FAOS skills as Gemini commands
cp skills/gemini/commands/faos-skill-*.toml .gemini/commands/

# Invoke with /faos-skill-{name}
```

### GitHub Copilot

```bash
# Copy all FAOS skills as Copilot instructions
cp skills/copilot/instructions/faos-skill-*.instructions.md .github/instructions/

# Skills auto-activate based on applyTo glob pattern
```

### Perplexity Computer

```bash
# Copy skills to your workspace
cp -r skills/perplexity/ .perplexity/skills/
```

## What's Inside

### Skills (520+)

| Category | Count | Examples |
|---|---|---|
| Engineering | 45+ | Dev workflows, code review, architecture, CI/CD |
| Product | 30+ | PRD creation, sprint planning, story writing |
| Data & AI | 40+ | Pipeline design, model cards, prompt engineering |
| Testing | 25+ | Test design, automation, ATDD, performance |
| C-Suite | 50+ | OKR review, board prep, financial review |
| Sales | 15+ | Discovery prep, deal qualification, proposals |
| Marketing | 15+ | Campaigns, ABM, content strategy, personas |
| Customer Service | 10+ | Ticket resolution, health review, playbooks |
| Creative | 10+ | Design thinking, storytelling, brainstorming |
| Industry Modules | 22 | FinTech, Healthcare, PropTech, Aviation, etc. |

### Agent Plugins (31)

Full persona agents with communication styles, KPIs, decision patterns, and vocabulary:

- **C-Suite**: CEO, CTO, CFO, CPO, COO, CRO, CMO, CHRO, CLO, CISO, CDAO
- **Engineering**: Developer, Architect, SRE, Security Engineer, AI Engineer
- **Product**: Product Manager, UX Designer, Scrum Master, Business Analyst
- **Data**: Data Engineer, Data Scientist, Data-AI Analyst, Data-AI Architect
- **GTM**: Sales Executive, Marketing Executive, Customer Service Manager
- **Specialized**: Tech Writer, Test Architect, Enterprise Architect

## Free vs FAOS Platform

| Feature | Free (This Repo) | FAOS Platform |
|---|---|---|
| 520+ portable skills | Yes | Yes |
| 31 agent plugins | Yes | Yes |
| Static skill instructions | Yes | Yes |
| **Dynamic ontology context** | - | Yes |
| **22 industry knowledge modules** | - | Yes |
| **Team context alignment** | - | Yes |
| **Ontology versioning & rollback** | - | Yes |
| **Real-time WebSocket sync** | - | Yes |
| **Custom industry modules** | - | Enterprise |
| **SSO/SAML** | - | Enterprise |

Skills give your AI instructions. **FAOS Ontology gives your AI understanding.**

## How It Works

```
┌──────────────────────┐     ┌───────────────────────┐
│  FAOS Skills (Free)  │     │  FAOS Ontology (Paid) │
│                      │     │                       │
│  "Write a PRD for    │     │  Role: Product Manager │
│   a SaaS product"    │     │  Domain: FinTech       │
│                      │     │  Industry: Banking     │
│  Static instructions │     │  KPIs: MRR, CAC, LTV  │
│  Same for everyone   │     │  Compliance: PCI-DSS   │
│                      │     │  Context: Your company │
└──────────┬───────────┘     └───────────┬───────────┘
           │                             │
           └──────────┬──────────────────┘
                      │
              ┌───────▼────────┐
              │   AI Output    │
              │                │
              │  Without FAOS: │
              │  Generic PRD   │
              │                │
              │  With FAOS:    │
              │  PRD that knows│
              │  your industry,│
              │  your KPIs,    │
              │  your team     │
              └────────────────┘
```

## Supported Platforms

| Platform | Skills | Plugins | Format | Ontology |
|---|---|---|---|---|
| Claude Cowork | Yes | Yes | SKILL.md | Via FAOS Extension |
| OpenAI Codex | Yes | - | SKILL.md + openai.yaml | Via FAOS Extension |
| Gemini CLI | Yes | - | TOML commands | Via FAOS Extension |
| GitHub Copilot | Yes | Yes | .instructions.md + .agent.md | Via FAOS Extension |
| Perplexity Computer | Yes | - | SKILL.md | Via FAOS Extension |
| VS Code (native) | - | - | - | Yes |

## Repository Structure

```
faos-skills-marketplace/
├── marketplace.json          # Marketplace metadata
├── skills/
│   ├── cowork/              # Claude Cowork format (SKILL.md)
│   │   ├── engineering/
│   │   ├── product/
│   │   ├── data-ai/
│   │   └── ...
│   ├── codex/               # OpenAI Codex format (SKILL.md + openai.yaml)
│   │   ├── engineering/
│   │   └── ...
│   ├── gemini/              # Gemini CLI format (TOML commands)
│   │   └── commands/
│   │       └── faos-skill-*.toml
│   ├── copilot/             # GitHub Copilot format (.instructions.md)
│   │   └── instructions/
│   │       └── faos-skill-*.instructions.md
│   └── perplexity/          # Perplexity Computer format (SKILL.md)
│       ├── engineering/
│       └── ...
├── plugins/                  # Claude Cowork agent plugins
│   ├── faos-ceo/
│   ├── faos-cto/
│   ├── faos-architect/
│   └── ...
├── .github/
│   └── workflows/
│       └── sync-from-faos.yml
└── README.md
```

## Contributing

Skills are auto-generated from the [FAOS Framework](https://github.com/frank-luongt/Foundation-AgenticOS). To contribute:

1. Add or modify skills in `.faos/skills/` in the source repo
2. Run `python scripts/export-skills.py --target all`
3. Submit a PR to the source repo

## License

Apache-2.0 — free to use, modify, and distribute.

---

**Built with [FAOS](https://faosx.ai)** — the enterprise agentic operating system that gives AI agents real business context.
