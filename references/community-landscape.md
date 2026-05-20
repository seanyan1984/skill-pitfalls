# Community Landscape: PITFALL Management in AI Agent Skills

## Hermes Agent (Official)

The official [creating-skills docs](https://hermes-agent.nousresearch.com/docs/developer-guide/creating-skills) define a `## Pitfalls` section in the standard SKILL.md format with a single line:

> "Known failure modes and how to handle them."

No categorization rules, no decision table format, no size limits, no append checklist.

## agentskills.io

The [best practices guide](https://agentskills.io/skill-creation/best-practices) introduces "Gotchas sections":

> "The highest-value content in many skills is a list of gotchas — environment-specific facts that defy reasonable assumptions."

Format: flat bullet lists. No categorization, no decision tables, no size limits.

## Claude Code Community

Jenny Ouyang's ["Stop Adding New Claude Skills — Fix the Broken Ones First"](https://buildtolaunch.substack.com/p/claude-skills-not-working-fix) (Substack, April 2026) is the most systematic skill quality article in the community. Her SSOT Audit focuses exclusively on **file-system-level** issues:

- **Broken Reference**: path points to a moved/deleted file
- **Orphan File**: skill with zero incoming references
- **Singleton**: flat `.md` instead of proper folder with SKILL.md
- **Duplicate Doctrine**: two files both defining how to do the same job
- **Dead Alias**: legacy command never deleted

**Skill content organization is a blind spot.** The entire audit operates at the file system layer — whether paths resolve, whether files are orphaned. What goes *inside* the skill, how pitfalls are structured, whether an agent can efficiently match symptoms to fixes — none of this is addressed.

## Tao An — "How to Write AI Skills That Don't Fail in Production"

[Medium article](https://tao-hpu.medium.com/how-to-write-ai-skills-that-dont-fail-in-production-6bb679897f30) (December 2025). Covers system-level reliability: cascading failures, structured output, permission boundaries, the autonomy spectrum. 26 academic/official references. Does not address knowledge document governance or pitfall organization.

## Anthropic — "Effective Context Engineering for AI Agents"

[Engineering blog](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) (2025). Discusses context window management, the "lost-in-the-middle" effect, and structured instructions vs prose. Relevant findings (numbered steps outperform prose, RFC 2119 keywords improve compliance) support the decision table approach, but do not explicitly address pitfall documentation structure.

## Gap Analysis

| Player | What They Cover | What's Missing |
|--------|----------------|----------------|
| Hermes official | `## Pitfalls` section header | No format spec, categorization, or size limits |
| agentskills.io | "Gotchas" as flat bullet lists | No decision tables or structural guidance |
| Claude Code community | File-system skill auditing | **Content organization is unexplored** |
| Production guides | System reliability patterns | Knowledge document governance |

**No existing framework addresses pitfall content organization within agent skills.** This is the gap this project fills.
