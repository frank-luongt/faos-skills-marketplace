# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in the FAOS Skills Marketplace, please report it
responsibly.

**Email:** security@faosx.ai

**What to include:**
- Description of the vulnerability
- Steps to reproduce
- Potential impact
- Suggested fix (if any)

**Response timeline:**
- Acknowledgment within 48 hours
- Assessment within 5 business days
- Fix or mitigation plan communicated within 10 business days

## Scope

This repository contains **AI skill instruction files** (plain text/markdown) and **agent persona
definitions** (YAML). It does not contain executable code, APIs, or services.

Security concerns for this repository are primarily:
- **Prompt injection** — skills that could cause unintended AI behavior
- **Credential exposure** — skills that inadvertently expose secrets or API keys
- **Harmful instructions** — skills that instruct AI to perform destructive actions

## Security Practices

- All skills are reviewed before merge for prompt injection risks
- No skills should reference hardcoded credentials, API keys, or secrets
- Skills must not instruct AI to bypass safety guardrails
- Agent plugins must not request elevated system permissions

## FAOS Platform Security

For security issues related to the **FAOS enterprise platform** (not this marketplace repo),
please contact security@faosx.ai with subject line "FAOS Platform Security."

## Supported Versions

| Version | Supported |
|---------|-----------|
| 1.x     | Yes       |
