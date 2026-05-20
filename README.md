# Skill Pitfalls

**Structured pitfall management for AI agent skills — categorization, decision tables, and A/B experiment validation.**

## Why This Exists

When you build AI agent skills (Hermes, Claude Code, etc.), you accumulate "gotchas" — known failure modes and how to handle them. Most people dump these into a flat bulleted list. Over time you get 30+ items with no structure, no searchability, and no diagnostic value.

This repo proposes a **structured approach**: categorize pitfalls, use decision tables for diagnostics, and validate the improvement with A/B experiments.

## Key Results

We ran A/B experiments comparing flat lists vs. structured pitfalls across 3 real error scenarios:

| Metric | Flat List (Old) | Structured (New) | Change |
|--------|-----------------|-------------------|--------|
| Diagnosis Accuracy (0-2) | 2.0 | 2.0 | = |
| Fix Accuracy (0-2) | 1.67 | 2.0 | **+20%** |
| Reasoning Steps | 3.6 | 2.7 | **-25%** |
| Overall Score | 2.67 | 3.67 | **+37%** |

**Key insight**: The biggest improvement comes when pitfalls are **scattered across multiple sections** (Scenario B: +2.0 points). Decision tables eliminate ambiguity — agents match symptoms to fixes in ~2 steps instead of linearly scanning N items.

## The Specification

See [`skill/SKILL.md`](skill/SKILL.md) for the full pitfall organization spec. Key rules:

1. **>5 items must be categorized** (anomaly diagnosis / DOM / anti-scraping / data / output / red-lines)
2. **Anomaly diagnostics must use decision tables** (symptom → diagnosis → fix)
3. **No duplication with body text** — pitfalls only cover blind spots
4. **Quality audit after refactoring** — 7-dimension checklist
5. **A/B validation optional but recommended** — see experiment protocol

## The Experiment

[`experiment/`](experiment/) contains the full A/B test:

- 3 scenarios (delegate timeout, CDP target loss, DB duplicate rows)
- 2 versions (flat list vs. categorized + decision tables)
- Multiple runs per version for statistical reliability
- Token efficiency analysis

## Community Landscape

This is a **gap** in the current AI agent ecosystem:

| Player | What They Cover | What's Missing |
|--------|----------------|----------------|
| Hermes official docs | `## Pitfalls` section header | No format spec, no categorization rules |
| [agentskills.io](https://agentskills.io/skill-creation/best-practices) | "Gotchas sections" as flat bullet lists | No decision tables, no size limits |
| [Claude Code community](https://buildtolaunch.substack.com/p/claude-skills-not-working-fix) | File-system-level skill auditing (broken refs, orphans, duplicates) | **Skill content organization is a blind spot** |
| [Production AI Skills guide](https://tao-hpu.medium.com/how-to-write-ai-skills-that-dont-fail-in-production-6bb679897f30) | System reliability (circuit breakers, structured output) | Knowledge document governance |

See [`references/community-landscape.md`](references/community-landscape.md) for detailed analysis.

中文说明：[`references/README_CN.md`](references/README_CN.md)

## Usage

### For Hermes Agent

Copy `skill/` to your Hermes skills directory:

```bash
cp -r skill/ ~/.hermes/skills/software-development/pitfall-organization/
```

### For Claude Code

Adapt the rules in `skill/SKILL.md` into your `.claude/skills/` or `CLAUDE.md` as a style guide for writing pitfalls.

### For Any Agent Framework

The categorization rules and decision table format are framework-agnostic. Apply them to any markdown-based skill/prompt documentation.

## Contributing

Issues and PRs welcome! Particularly interested in:

- Pitfall patterns from other agent frameworks (LangChain, CrewAI, AutoGen)
- Additional A/B test scenarios
- Translations of the spec

## License

MIT
