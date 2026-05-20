## 9. PITFALLS

### 🚨 执行超时与降级

| 症状 | 判断方法 | 修复方案 |
|------|----------|----------|
| delegate_task 600s 超时，未写出文件 | 检查 output 中是否有 write_file 调用 | **不要重试同配置**。立即拆成 1本/subagent 重跑失败批次 |
| 同一本书超时 ≥3 次 | 连续 3 次 delegate_task 超时 | 主会话直接 read_file + write_file（主会话无单任务倒计时，是最终 fallback） |
| 2本/subagent 偶尔成功偶尔超时 | 不可预测（章节长度+内容密度决定） | **默认用 1本/subagent**，2本只在已知章节短（<2.5K字/章）时考虑 |
| execute_code 输出被截断 | 20章正文返回 ~14K 而非完整 ~68K | 分批读取，每批 ≤ 10 章 |

**关键参数**：
- 智谱 API 并发上限 = 2，batch 并发不超过 2
- 分析密度因维度而异：金手指稀疏（2本稳），人设密集（2本可能爆），不能跨维度套用耗时经验
- 内容密度比文件大小更影响超时（多线叙事 > 单线叙事）

### 文件读取与数据

- **预提取正文到 raw/*.txt 是必须步骤**：子 agent 直接查 DB（20+ 次 SQL）容易耗尽超时。预提取后只需 2-3 次 read_file
- **read_file 每次读 1000 行，不是 500 行**：prompt 中必须告知子 agent 文件总行数和分段策略
- **read_file 在 execute_code 中必须带 offset=1**：不带 offset 返回空 content
- **read_file 有去重机制**：execute_code 中同一文件第二次读取返回空。解决方案：用 `terminal('cat path')` 代替
- **execute_code 变量不跨脚本保留**：每次是独立 Python 进程，需要多步处理的任务放一个脚本里或通过文件 I/O 传递
- **大文件合并用「Python 搬运 + LLM 推理」模式**：input >30KB 时，execute_code 中用 Python 做文本拆分组装，LLM 只做推理判断和 write_file
- **books 表缺记录**：先查 rank_snapshot 表补回，prompt 中提醒"直接用 book_id 查 chapters"兜底

### 执行编排

- **delegate_task batch 是首选方案**，cron 链式不可靠（agent 跳过"标记完成+创建下一个 cron"导致断链）
- **汇总分两级**：批摘要（input <20KB）可以 delegate；总报告（input >50KB）必须在主会话做
- **文风维度不需要读全文**：5 章样本足够（风格高度稳定），预提取到 raw_sampleA/B/C/
- **市场缺口维度不需要读正文**：输入是前 4 维度总报告的合并文件（~54KB），子 agent 只需 `["file"]` 工具集

### 维度边界纪律

- 每个维度只提取本维度内容，严禁跨维度混入。prompt 末尾必须加维度边界声明（见 §8.5）
- 维度完成后必须做质量门禁：抽查单片段 + 总报告 scope 检查 + 发现问题先修再往下

### 🚨 红线（不可违反）

- **【红线】cron job 删除/暂停必须用户明确授权**，不能自作主张
- **【红线】中断后恢复必须用户确认**：不能把用户的提问/讨论理解为批准执行。上下文过长时尤其容易忘记——先问"用户确认了吗？"
- **不要自作主张换方案**：遇到问题先排查根因，停下来问用户
- **查文档优先于读代码**：Hermes 功能问题先查官方文档和 GitHub issues

### Cron 相关（仅备用）

- cron schedule 必须用时间戳格式（如 `2026-05-04T11:40:00`），不支持 `+4m` 相对格式
- 时间余量至少 +3min，避免 next_run 已过
- tick lock 阻塞是已知问题（Issue #3752），一个慢 job 独占整个 scheduler
- 链式 cron 不可靠：Hermes 无心跳/链式监控（Issue #15400 未实现），链断了只能靠主 agent 发现
- `hermes cron create` 正确语法：schedule 和 prompt 是位置参数，不是 flag

### 其他

- 文件命名用「批N」（如 `AgentA_批1.md`），不是「段」。1书/subagent 时用 `AgentA_{书名}.md`
- 同一 prompt 的 A/B/C agent 产出已验证不同（LLM 随机性足够），不需要为不同 agent 设计不同视角 prompt
- 恢复长会话时必须验证实际文件（`ls -lt`），不能仅凭上一条飞书消息的状态标注判断进度
- Claude Code ACP 做 delegate_task 子 agent 需要设 `HERMES_COPILOT_ACP_COMMAND` 环境变量（nvm 路径不在 gateway PATH 中）
