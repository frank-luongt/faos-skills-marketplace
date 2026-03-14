# Gemini Deep Research — Architecture Patterns Reference

This document captures key patterns from Google's Gemini Deep Research Agent that inform our
deep-research skill design.

## Core Architecture

Gemini Deep Research (powered by Gemini 3.1 Pro) operates through an **iterative plan-search-read
loop** with autonomous decision-making at each step.

### The Iterative Research Loop

```
Query → Plan → [Search → Read → Reason → Adapt Plan] × N → Synthesize → Self-Critique → Report
```

1. **Planning** — formulates research strategy, breaks problem into sub-tasks
2. **Searching** — conducts web searches, identifies relevant sources
3. **Reading** — extracts and analyzes information from sources
4. **Reasoning** — evaluates findings, identifies gaps, decides next move
5. **Adaptation** — refines plan based on what's been learned
6. **Synthesis** — compiles findings into comprehensive report
7. **Self-critique** — multiple passes to enhance clarity, accuracy, and detail

### Key Design Decisions

| Decision | Gemini's Approach | Our Adaptation |
|----------|-------------------|----------------|
| Execution model | Asynchronous (background) | Foreground with progress updates |
| Tool access | Web search + file search only | WebSearch + WebFetch + sub-agents |
| Planning | Internal (opaque) | Visible plan shared with user |
| Iteration control | Autonomous (up to 60 min) | Autonomous with stop criteria |
| Self-critique | Multi-pass internal | Explicit 3-pass critique protocol |
| Output format | Markdown with citations | Structured report with confidence ratings |

## Performance Benchmarks (Gemini Deep Research)

| Benchmark | Score | What It Tests |
|-----------|-------|---------------|
| HLE (Humanity's Last Exam) | 46.4% | Cross-domain expert reasoning |
| DeepSearchQA | 66.1% | 900 causal chain tasks across 17 fields |
| BrowseComp | 59.2% | Complex web browsing and information synthesis |

## What Makes Deep Research Different from Standard Search

1. **Multi-step reasoning** — each search builds on prior findings
2. **Autonomous gap identification** — the system knows what it doesn't know
3. **Source triangulation** — critical claims verified across multiple sources
4. **Adaptive strategy** — research plan evolves as information emerges
5. **Quality self-assessment** — output is critiqued before delivery

## Cost & Resource Patterns

| Complexity | Searches | Tokens | Estimated Cost | Duration |
|-----------|----------|--------|---------------|----------|
| Standard | ~80 | ~250K cached input | $2-3 | 5-15 min |
| Complex | ~160 | ~500K cached input | $3-5 | 15-30 min |
| Exhaustive | ~300+ | ~1M+ cached input | $5-10 | 30-60 min |

## Limitations of Gemini's Approach (and How We Differ)

| Gemini Limitation | Our Approach |
|-------------------|-------------|
| No custom tools or MCP | Full Claude Code toolchain available |
| No plan approval workflow | User sees plan before execution |
| Public web sources only | Can access local files, codebases, internal docs |
| Opaque reasoning | Visible gap analysis and iteration logs |
| No structured output | Configurable report templates |
| Single-agent execution | Multi-agent with parallel sub-agents |

## Sources

- [Gemini Deep Research API Docs](https://ai.google.dev/gemini-api/docs/deep-research)
- [Gemini Deep Research Overview](https://gemini.google/overview/deep-research/)
- [Build with Gemini Deep Research](https://blog.google/technology/developers/deep-research-agent-gemini-api/)
- [Anthropic Multi-Agent Research System](https://www.anthropic.com/engineering/built-multi-agent-research-system)
