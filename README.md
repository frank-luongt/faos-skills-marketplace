# FAOS Skills Marketplace

> **526+ AI-powered skills and 31 agent plugins** for Claude Cowork, OpenAI Codex, Gemini CLI, GitHub Copilot, and Perplexity Computer.

The largest open-source AI skills marketplace. Built by the [FAOS Framework](https://faosx.ai) team.

**Apache 2.0 Licensed** — free to use, modify, and distribute commercially.

## Quick Install

### Claude Cowork

```bash
# Install a single skill (e.g., code-review)
cp skills/cowork/code-reviewer/SKILL.md .claude/skills/

# Install all skills
cp -r skills/cowork/*/SKILL.md .claude/skills/

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

### Skills (526+) — [Browse the full catalog](CATALOG.md)

| Category | Count | Examples |
|---|---|---|
| Engineering | 180+ | Dev workflows, code review, architecture, CI/CD, testing |
| Product & Strategy | 90+ | PRD creation, sprint planning, OKR review, roadmap |
| Growth & Sales | 80+ | Deal qualification, proposals, campaigns, ABM |
| Data & AI | 60+ | Pipeline design, EDA, model cards, prompt engineering, dashboards |
| Operations & Leadership | 70+ | Board prep, financial review, talent review, security posture |
| Creative & Design | 38+ | Brainstorming, design thinking, storytelling, wireframes, diagrams |

### Agent Plugins (31)

Full persona agents with communication styles, KPIs, decision patterns, and vocabulary:

- **C-Suite (11):** CEO, CTO, CFO, CPO, COO, CRO, CMO, CHRO, CLO, CISO, CDAO
- **Engineering (6):** Developer, Architect, SRE, Security Engineer, Enterprise Architect, Solo Dev
- **Product (3):** Product Manager, UX Designer, Business Analyst
- **Data & AI (5):** AI Engineer, Data Engineer, Data Scientist, Data-AI Architect, Data-AI Analyst
- **GTM (3):** Sales Executive, Marketing Executive, Customer Service Manager
- **Delivery (2):** Scrum Master, Tech Writer
- **Quality (1):** Test Architect

### Supported Platforms

| Platform | Skills | Plugins | Format |
|---|---|---|---|
| Claude Cowork | 526 | 31 | SKILL.md |
| OpenAI Codex | 526 | - | SKILL.md + openai.yaml |
| Gemini CLI | 526 | - | TOML commands |
| GitHub Copilot | 526 | - | .instructions.md |
| Perplexity Computer | 526 | - | SKILL.md |

## Skills Marketplace vs FAOS Platform

| Feature | Marketplace (Free) | FAOS Platform |
| --- | --- | --- |
| 526+ portable skills | Yes | Yes |
| 31 agent plugins | Yes | Yes |
| 5 platform formats | Yes | Yes |
| Creative extension (brainstorming, design thinking, storytelling) | Yes | Yes |
| Apache 2.0 license | Yes | Yes |
| 7-Phase Execution Engine | - | Yes |
| 3-Layer Memory Architecture | - | Yes |
| Ontology Engine + 22 Industry Blueprints | - | Yes |
| Agent Autonomy Engine (L0-L3) | - | Yes |
| Governance Gates & RBAC | - | Yes |
| Per-Agent Cost Controls & Budgets | - | Yes |
| Portal, Desktop & Mobile Apps | - | Yes |
| Multi-Tenant Operations | - | Yes |
| Enterprise Support & SLA | - | Yes |

Skills give your AI instructions. **FAOS gives your AI understanding.**

[Learn more about the platform](https://faosx.ai) | [Request access](https://faosx.ai/request-access)

## Repository Structure

```
faos-skills-marketplace/
├── skills/
│   ├── cowork/              # Claude Cowork — 526 SKILL.md files (flat)
│   │   ├── code-review/SKILL.md
│   │   ├── dev-story/SKILL.md
│   │   ├── prd-creation/SKILL.md
│   │   └── ... (526 skill directories)
│   ├── codex/               # OpenAI Codex — SKILL.md
│   ├── gemini/              # Gemini CLI — TOML commands
│   ├── copilot/             # GitHub Copilot — .instructions.md
│   └── perplexity/          # Perplexity Computer — SKILL.md
├── plugins/                 # 31 agent plugins
│   ├── faos-ceo/
│   ├── faos-cto/
│   ├── faos-architect/
│   └── ... (31 plugin directories)
├── .claude-plugin/          # Claude Cowork marketplace manifest
├── marketplace.json         # Registry metadata
├── CONTRIBUTING.md          # How to contribute
├── SECURITY.md              # Security policy
├── CHANGELOG.md             # Release history
├── LICENSE                  # Apache 2.0
└── README.md
```

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

**Quick version:**

1. Fork this repo
2. Add or improve skills in `skills/cowork/{category}/`
3. Submit a PR — we'll handle cross-platform exports

## Security

Found a security issue? See [SECURITY.md](SECURITY.md) for our responsible disclosure process.

## License

Apache 2.0 — see [LICENSE](LICENSE) for full text.

---

**Built with [FAOS](https://faosx.ai)** — the enterprise agentic operating system.
