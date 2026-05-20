---
name: pitfall-organization
description: Use when writing, appending, refactoring, or auditing PITFALL sections in AI agent skills. Enforces categorization, decision table format, and deduplication rules to keep pitfall documentation actionable.
version: 1.0.0
author: seanyan
license: MIT
metadata:
  hermes:
    tags: [skill, pitfall, documentation, quality, structure]
    related_skills: [hermes-agent-skill-authoring]
---

# Pitfall Organization for Agent Skills

## Overview

When you maintain AI agent skills, you accumulate known failure modes. Most people dump these into a flat bullet list. Over time you get 30+ unstructured items — no categories, no priority, no diagnostic value. When an agent encounters an error, it needs **symptom → diagnosis → fix** decision paths, but can't extract them from a wall of text.

This skill defines rules for organizing PITFALL sections so that agents can quickly match symptoms to fixes. It works with any markdown-based skill format (Hermes, Claude Code, etc.).

## When to Use

- Adding a new pitfall item to any agent skill
- Refactoring an existing flat pitfall list into structured form
- Auditing skill quality and finding fragmented pitfalls
- Reviewing whether a pitfall section has grown too large

Don't use for:
- General skill authoring (use `hermes-agent-skill-authoring` instead)
- Runtime error handling logic (this is about documentation, not code)

## Categorization

PITFALL sections exceeding **5 items must be categorized**. The specific category names depend on your domain, but use this priority structure:

| Priority | What Goes Here | Example Categories |
|----------|---------------|-------------------|
| **Highest** | Symptom → diagnosis → fix decision tables | Anomaly Diagnosis, Error Recovery |
| High | Tool-specific gotchas, API quirks | Tool Traps, Integration Gotchas |
| Medium | Data formats, parameter edge cases | Data & Parameters, Input Validation |
| Low | Project conventions, naming rules | Project Conventions, Style Rules |

Not all categories are required. The key principle: **anomaly diagnosis always goes first**, because that's what agents need when something breaks.

## Decision Tables

Anomaly diagnosis items **must** use decision table format. Do not write prose narratives.

### Format

```
| Symptom | Diagnosis | Fix |
|---------|-----------|-----|
| Specific observable symptom | One executable diagnostic step | Specific command or operation |
```

### Why

Prose is hard for agents to match against. A decision table lets an agent scan the Symptom column, find a match, and read the Fix column — typically in 1-2 steps instead of linearly scanning an entire flat list.

### Example

```
| Symptom | Diagnosis | Fix |
|---------|-----------|-----|
| delegate_task 600s timeout, no file written | Check output for write_file calls | Don't retry same config. Split to 1 item/subagent |
| Same item times out ≥3 times | Consecutive delegate_task timeouts | Main session reads and writes directly (no per-task timer) |
```

## Writing Single Items

Each pitfall item should follow this pattern:

- **One sentence**: `Symptom → Cause or diagnosis → Fix`
- **Must include an executable action**: specific command, selector, or parameter value
- **No storytelling**: avoid "In one session we discovered..."
- **No duplicating the skill body**: pitfalls cover blind spots, not tutorials

### Bad

> When running the collector, sometimes chapters get duplicated because the table doesn't have a unique constraint and INSERT OR REPLACE doesn't work as expected with auto-increment IDs.

### Good

> **chapters table lacks (book_id, chapter_num) UNIQUE constraint**: `INSERT OR REPLACE` inserts duplicate rows (auto-increment `id` never conflicts). Fix: `CREATE UNIQUE INDEX idx ON chapters(book_id, chapter_num)` or use `fill_one_book.py` which checks before inserting.

## Pre-Append Checklist

Before adding any item to a PITFALL section, run through this list:

1. **Is the existing PITFALL categorized?** → If not, categorize first, then append
2. **Which category does the new item belong to?** → Place it under that category
3. **Is it anomaly diagnosis?** → Must use decision table format, placed at the top
4. **Does it duplicate an existing item?** → Merge into the existing item, don't append a new one
5. **Does total exceed 5 × category count?** → Consider splitting to `references/pitfalls.md`

## Deduplication with Body Text

This rule applies to **operation-manual-style skills** — ones with detailed step-by-step instructions and inline warnings (⚠️). For reference-style or API-doc-style skills where the body doesn't include inline warnings, all error knowledge goes in the pitfall section.

**When applicable**, the most common refactoring mistake is: copying warnings from the skill body into PITFALL, becoming an echo chamber.

**Rules**:
- If the body already covers a gotcha right next to the relevant operation step (typically marked with ⚠️), the PITFALL section should give **one line of cross-reference** only — not repeat the full explanation
- PITFALL should only cover **blind spots the body doesn't address**: implicit pitfalls, cross-scenario pitfalls, or places where the body warning isn't actionable enough
- After refactoring, audit the body-text duplication rate. If **>50%** of PITFALL items are just rehashing the body, the PITFALL section adds no value

## Handling Growth

When PITFALL exceeds **20 items**:

1. Keep anomaly diagnosis tables in `SKILL.md` (highest priority — agents read these first)
2. Move lower-priority categories to `references/pitfalls.md`
3. In `SKILL.md`, leave one-line pointers: `See references/pitfalls.md for [category name] pitfalls.`

## Common Pitfalls

1. **Writing prose instead of decision tables.** Agents can't efficiently match "sometimes things break because..." against a concrete error. Use the Symptom/Diagnosis/Fix table.

2. **Copying body text into PITFALL (for operation-manual skills).** If the body already says "⚠️ sleep 5 seconds between requests", the PITFALL doesn't need to say it again. Cross-reference or skip. Note: this only applies when the skill body has inline warnings.

3. **Appending without categorizing.** The 6th uncategorized item is a violation. Categorize first.

4. **Vague symptoms.** "CDP stops working" is not actionable. "Runtime.evaluate returns 'No target with given id found' but Target.getTargets shows target still exists" is.

5. **Not merging duplicates.** Two items describing the same root cause with slightly different wording will confuse agents. Merge them.

## Verification Checklist

- [ ] >5 items are categorized with specific category names
- [ ] Anomaly diagnosis items use decision tables (Symptom / Diagnosis / Fix)
- [ ] No uncategorized paragraph exceeds 5 items
- [ ] No duplicate items (same root cause, different wording)
- [ ] Body-text duplication <50% (PITFALL has incremental value)
- [ ] No prose narratives ("in one session we found...")
- [ ] All original items from pre-refactor are covered (no information loss)
