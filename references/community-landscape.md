# 社区调研：PITFALL 治理的社区现状（2026-05-20）

## Hermes 官方

官方 creating-skills 文档规定的 SKILL.md 标准格式里有 `## Pitfalls` 章节，说明只有一句话：
> "Known failure modes and how to handle them."

没有分类规范、没有决策表格式、没有条数上限、没有追加检查清单。

## agentskills.io

最佳实践文档有 "Gotchas sections" 的概念：
> "The highest-value content in many skills is a list of gotchas — environment-specific facts that defy reasonable assumptions."

但格式就是平铺 bullet list，没有分类、决策表、条数上限。

## Claude Code 社区

Jenny Ouyang "Stop Adding New Claude Skills — Fix the Broken Ones First" (Substack, 2026-04-02)

SSOT Audit 关注的全是文件系统层面：
- Broken Reference（路径断了）
- Orphan File（没人调用的 skill）
- Singleton（平铺 .md 没有文件夹包裹）
- Duplicate Doctrine（两个文件说同一件事）
- Dead Alias（遗留命令没删）

skill 内部内容质量（包括 Pitfalls 怎么写）完全没覆盖。

## Tao An "How to Write AI Skills That Don't Fail in Production" (Medium, 2025-12-16)

聚焦系统级可靠性（级联失败、结构化输出、权限边界）。不涉及知识文档治理。

## 结论

没有任何人提出过"pitfall 分类规范 + 决策表 + 追加检查清单"这一套。skill 内部内容组织是社区盲区。
