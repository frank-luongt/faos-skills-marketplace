# Changelog

All notable changes to the FAOS Skills Marketplace will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [1.0.0] - 2026-04-07

### Added

- **536 AI skills** across 6 categories: Engineering, Product & Strategy, Growth & Sales, Data & AI, Operations & Leadership, Creative & Design
- **5 platform exports:** Claude Cowork, OpenAI Codex, Gemini CLI, GitHub Copilot, Perplexity Computer
- **42 agent plugins** with full persona definitions (C-Suite, Engineering, Product, Data, GTM, Delivery, Quality, Creative)
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
