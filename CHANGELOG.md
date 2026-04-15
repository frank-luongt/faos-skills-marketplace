# Changelog

All notable changes to the FAOS Skills Marketplace will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.1.0] - 2026-04-15

### Added

- **400+ new skills** across all platforms — total now 930+ unique skills
- Expanded cloud coverage: 120+ Azure, AWS, GCP, Databricks, Snowflake skills
- Expanded security coverage: 60+ skills (OWASP, NIST, SOC2, HIPAA, PCI-DSS, threat modeling)
- Expanded AI/ML coverage: 100+ skills (LangChain, RAG, fine-tuning, vector DBs, MLOps)
- 11 Creative extension agent personas (brainstorming coach, storyteller, design thinking, innovation strategist, problem solver, presentation master)

### Changed

- Updated README with accurate skill counts per platform
- Updated marketplace.json with per-platform counts (932 Cowork, 923 Codex, 923 Perplexity, 518 Gemini, 518 Copilot)
- Improved skill category breakdown (now 12+ categories)
- Renamed "Claude Cowork" references to "Claude Code" across all files
- Replaced Discord references with GitHub Discussions

### Fixed

- Pre-launch security and accuracy fixes from executive review
- Remaining risk items from executive review

## [1.0.0] - 2026-04-07

### Added

- **526 AI skills** across 6 categories: Engineering, Product & Strategy, Growth & Sales, Data & AI, Operations & Leadership, Creative & Design
- **5 platform exports:** Claude Code, OpenAI Codex, Gemini CLI, GitHub Copilot, Perplexity Computer
- **31 agent plugins** with full persona definitions (C-Suite, Engineering, Product, Data, GTM, Delivery, Quality, Creative)
- **22 industry module skills:** FinTech, Healthcare, PropTech, Aviation, EduTech, Retail, Manufacturing, Insurance, Logistics, Gaming, Agriculture, Automotive, Banking, Travel, MarTech, Cybersecurity, Talent & HR, Professional Services, Systems Integration, Software, Investment Management
- Apache 2.0 license
- Marketplace metadata (`marketplace.json`)
- GitHub Actions workflow for automated sync from FAOS source

### Platform Formats

- **Cowork:** `SKILL.md` with YAML frontmatter (name, description, tags)
- **Codex:** `SKILL.md` + `agents/openai.yaml` for UI metadata
- **Gemini:** `.toml` command files in `.gemini/commands/`
- **Copilot:** `.instructions.md` with `applyTo` glob patterns in `.github/instructions/`
- **Perplexity:** `SKILL.md` with subagent annotations
