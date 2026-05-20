---
name: pitfall-organization
description: Use when writing, refactoring, or auditing PITFALL sections in AI agent skills. Defines categorization rules, decision table format, and quality checklist.
tags: [skill, pitfall, quality, structure, agent]
---

# PITFALL Organization Specification

## The Problem

When building AI agent skills, you accumulate known failure modes ("pitfalls"). Most people dump these into a flat bulleted list. Over time you get 30+ items with no structure. When an agent encounters an error, it needs **symptom → diagnosis → fix** decision trees, but can't find them in a fragmented list.

**No existing framework addresses this.** Agent skill quality efforts focus on file-system hygiene (broken references, orphan files). Skill content organization remains unexplored.

## Core Rules

### 1. Categorize When >5 Items

PITFALL sections exceeding 5 items **must** be categorized. Use these categories as needed:

| Category | Content | Priority |
|----------|---------|----------|
| **🚨 Anomaly Diagnosis** | Symptom → diagnosis → fix decision tables | **Highest** — always first |
| **Selector / DOM Traps** | CSS selectors, DOM operations, framework quirks | High |
| **Anti-Scraping / Rate Limits** | Rate limit codes, pacing, ban handling | High |
| **Data & Parameters** | Parameter mapping, data formats, edge cases | Medium |
| **Output & Rendering** | Platform rendering issues, format constraints | Medium |
| **Project Conventions** | Directory structure, naming, process discipline | Low |

### 2. Decision Tables for Diagnostics

Anomaly diagnosis items **must** use decision table format:

```
| Symptom | Diagnosis | Fix |
|---------|-----------|-----|
| Specific observable symptom | One executable diagnostic step | Specific command or operation |
```

**Do not write prose.** Agents forget prose. Decision tables enable direct symptom matching.

Example:

```
| Symptom | Diagnosis | Fix |
|---------|-----------|-----|
| delegate_task 600s timeout, no file written | Check output for write_file calls | Don't retry same config. Split to 1 book/subagent |
| Same book times out ≥3 times | Consecutive delegate_task timeouts | Main session read_file + write_file (no per-task timer) |
```

### 3. Pre-Append Checklist

Before adding any item to a PITFALL section:

1. **Is the existing PITFALL categorized?** → Categorize first, then append
2. **Which category does the new item belong to?** → Place under that category
3. **Is it anomaly diagnosis?** → Must use decision table format, placed first
4. **Does it duplicate an existing item?** → Merge, don't append
5. **Does total exceed 5 × category count?** → Consider splitting to `references/` file

### 4. Single-Item Writing Rules

- **One sentence**: `Symptom → Cause/Diagnosis → Fix`
- **Must include executable action**: specific command, selector, or parameter value
- **No storytelling**: avoid "In one session we discovered..."
- **No duplicating skill body text**: pitfalls are blind-spot records, not tutorials

### 5. Deduplication with Body Text

The most common refactoring mistake: copying ⚠️ warnings from the body into PITFALL, becoming an echo chamber.

**Rules**:
- Body text already covers near an operation step (marked with ⚠️) → PITFALL gives one-line cross-reference only
- PITFALL only covers **blind spots the body doesn't address**: implicit pitfalls, cross-scenario pitfalls, insufficiently actionable body warnings
- After refactoring, audit body-text duplication rate. >50% means PITFALL adds no value

### 6. Handling Growth

When PITFALL exceeds 20 items:
- Keep anomaly diagnosis in SKILL.md (highest priority for agents to read)
- Move other categories to `references/pitfalls.md`, SKILL.md keeps one-line pointer

### 7. Quality Audit

After refactoring, validate with this checklist:

| Check | Passing Standard |
|-------|-----------------|
| Categorization | >5 items categorized with specific category names |
| Decision tables | Anomaly diagnosis has 3-column tables (symptom/diagnosis/fix) |
| Flat lists | No uncategorized paragraph exceeding 5 items |
| Internal duplication | No duplicate items |
| Body-text duplication | <50%, has incremental value |
| Prose narratives | No "in one session" style narratives |
| Information loss | All original items covered (compare against pre-refactor file) |

**Rating**: ✅ Pass / ⚠️ Minor fix needed / ❌ Major rework

### 8. A/B Validation (Optional)

After refactoring + quality audit, run an A/B experiment to quantify the improvement:

1. Take 2-3 typical error scenarios from pre-refactor and post-refactor PITFALLs
2. Construct isolated contexts: one group gets only old PITFALL, another gets only new
3. Give identical error description prompts to each group
4. Score on: diagnosis accuracy (0-2), fix accuracy (0-2), wrong suggestions (N), reasoning steps (N)

**Scoring**: `Overall = Diagnosis + Fix - WrongSuggestions - (Steps > 3 ? 1 : 0)`

**Our results** (3 scenarios × 2 versions):
- Overall score: 2.67 → 3.67 (+37%)
- Fix accuracy: 1.67 → 2.0 (+20%)
- Biggest improvement when pitfalls were scattered across multiple sections (+2.0 points)

## When to Use This Spec

- Adding pitfall items to any agent skill
- Refactoring existing flat pitfall lists
- Auditing skill quality and finding fragmented pitfalls
- Validating refactored pitfalls with A/B experiments
