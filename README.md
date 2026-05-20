[English](#skill-pitfalls) | [简体中文](references/README_CN.md)

# Skill Pitfalls

**Structured pitfall management for AI agent skills — categorization, decision tables, and validation.**

When you build AI agent skills, you accumulate known failure modes. Most people dump these into a flat bullet list. Over time you get 30+ items with no structure — agents can't match symptoms to fixes. This project provides a specification and experimental validation for organizing pitfalls into actionable decision tables.

## Results

A/B experiments comparing flat lists vs. structured pitfalls (3 scenarios, multiple runs):

| Metric | Flat List | Structured | Change |
|--------|-----------|------------|--------|
| Overall Score | 2.67 | 3.67 | **+37%** |
| Fix Accuracy | 1.67 | 2.0 | **+20%** |
| Reasoning Steps | 3.6 | 2.7 | **-25%** |

Biggest improvement when pitfalls were scattered across multiple sections (+2.0 points).

## Contents

| Section | Description |
|---------|-------------|
| [`skill/SKILL.md`](skill/SKILL.md) | The full specification — categorization rules, decision table format, writing guidelines, checklist |
| [`experiment/`](experiment/) | A/B test data: prompts, old vs. new pitfalls, results, token analysis |
| [`references/community-landscape.md`](references/community-landscape.md) | Why this matters — community gap analysis |

## The Spec in 5 Rules

1. **>5 items → categorize.** Use: anomaly diagnosis (highest priority), selector/DOM traps, anti-scraping, data & params, output & rendering, project conventions.

2. **Anomaly diagnosis → decision table.** Three columns: `Symptom | Diagnosis | Fix`. No prose.

3. **Don't duplicate the body.** If the skill body already covers a gotcha (⚠️ marker), the PITFALL gives a one-line cross-reference only.

4. **Pre-append checklist.** Every new item: categorize → pick category → decision table if diagnostic → merge if duplicate → split if too many.

5. **Quality audit.** 7-dimension checklist after refactoring. Pass / minor fix / rework.

## Quick Start

**Hermes Agent:**
```bash
cp -r skill/ ~/.hermes/skills/software-development/pitfall-organization/
```

**Claude Code / Others:** The rules in `skill/SKILL.md` are framework-agnostic. Apply them to any markdown-based skill or prompt documentation.

## Community Context

Current skill quality efforts focus on **file-system hygiene** (broken references, orphan files). **Content organization within skills** remains unexplored.

| Player | Covers | Missing |
|--------|--------|---------|
| Hermes official | `## Pitfalls` section header | No format spec |
| agentskills.io | Flat "gotchas" bullet lists | No decision tables |
| Claude Code community | File-system skill auditing | Content structure is a blind spot |

See [`references/community-landscape.md`](references/community-landscape.md) for full analysis.

## Contributing

Issues and PRs welcome — especially pitfall patterns from other agent frameworks and additional test scenarios.

## License

MIT
