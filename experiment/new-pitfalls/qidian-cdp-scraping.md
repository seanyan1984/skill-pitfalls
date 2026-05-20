## PITFALLS

### CDP 执行

- **必须用 foreground terminal**：Playwright 的 Node.js 子进程 stdout 在 background 模式下被吞掉，必须 `python3 -u script.py 2>&1`
- **每个 Playwright 操作用 `context.new_page()` + `page.close()`**，不复用已有 page
- 不要用 curl/wget 带 cookie 访问，httpOnly 字段缺失会被 WAF 拦截。CDP 共享浏览器登录态是唯一可靠方案
- VIP 章节需付费，免费试读通常覆盖前几十章（够开局分析）

### 数据解析

- **`g_data.pageJson` 含注释**：不能直接 `JSON.parse()`，需先 strip 注释
- **章节标题 `h1` 含评论数**：需清理尾部数字；正文每段末尾也可能附评论数
- **Python triple-quoted string 嵌 JS**：`\\n` 会被 Python 吃掉，用 `String.fromCharCode(10)` 替代

### 数据库

- **chapters 表缺 (book_id, chapter_num) UNIQUE 约束**：`INSERT OR REPLACE` 会插入重复行（id 是自增主键）。补采必须用 `fill_one_book.py` 先查已有章节再插入
- **`--category` 参数别名**：既接受中文名（`xianxia`）也接受 URL 后缀（`chn22`），dict 里两种 key 都要映射
- 起点章节 ID 全局递增，可横向对比不同书入库先后

## 批量补采策略

需要为多本书补采章节时（如 top10 全部补到 20 章），**不要试图用一个脚本从头跑到尾**。正确做法：

### 1. 拆成独立的单次 cron job

每本书一个一次性 cron job，间隔 1 小时（防反爬）。好处：天然隔离、stdout 不丢、单个失败不影响后续。

```
单本补采脚本: ~/Documents/project-mini/scripts/fill_one_book.py
用法: python3 -u scripts/fill_one_book.py --book-id <ID> --target 20
```

脚本只插入缺失章节（幂等），重复运行不会产生重复数据。

### 2. 采集前先查目录上限

目标章节数（如 20）是需求，但实际可采数取决于作者写了多少。新书可能才写 3-13 章。应先查 `get_chapter_urls()` 返回的 `total`，动态调整期望值，避免采集结果与预期不符时困惑。