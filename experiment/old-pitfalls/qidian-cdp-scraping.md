## PITFALLS

- **CDP 采集必须用 foreground 模式**：Playwright 的 Node.js 子进程 stdout 在 background 模式下会被吞掉，**必须用 foreground terminal** 运行（`python3 -u script.py 2>&1`）
- **分类参数别名**：`--category` 既接受中文名（`xianxia`）也接受 URL 后缀（`chn22`），dict 里两种 key 都要映射
- 起点反爬强（WAF + 验证码），CDP 共享浏览器登录态是唯一可靠方案
- 不要用 curl/wget 带 cookie 访问，httpOnly 字段缺失会被 WAF 拦截
- VIP 章节需付费，免费试读通常覆盖前几十章（够开局分析）
- 章节标题 `h1` 含评论数，需清理尾部数字；正文每段末尾也可能附评论数
- Python triple-quoted string 嵌 JS 代码时，`\\n` 会被 Python 吃掉，用 `String.fromCharCode(10)` 替代
- `g_data.pageJson` 含注释不能直接 `JSON.parse()`，需先 strip 注释
- 起点 chapterId 全局递增，可横向对比不同书入库先后
- 每个 Playwright 操作用 `context.new_page()` + `page.close()`，不复用已有 page
- **chapters 表缺 (book_id, chapter_num) UNIQUE 约束**：直接用 collector v2 的 `INSERT OR REPLACE` 会插入重复行（因为 id 是自增主键）。补采必须用 `fill_one_book.py` 先查已有章节再插入，或手动给表加唯一约束
