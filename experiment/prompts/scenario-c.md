# 场景 C：chapters 表重复行（qidian-cdp-scraping）

## 错误描述

我用 collector v2 的 INSERT OR REPLACE 补采了一些章节。

之后查询发现 chapters 表里同一本书同一章节号有多行记录。比如：

```
sqlite3 qidian.db "SELECT id, book_id, chapter_num FROM chapters WHERE book_id='1048579926' AND chapter_num=7"
```

返回了 3 条记录，id 分别是 45、128、203。

请帮我诊断原因和修复方案。

## 期望的诊断

根因：chapters 表缺 (book_id, chapter_num) 的 UNIQUE 约束。collector v2 用 INSERT OR REPLACE，但因为 id 是自增主键而非 (book_id, chapter_num) 组合主键，REPLACE 条件匹配不到已有行，所以每次都插入新行。

期望修复：
1. 用 fill_one_book.py 补采（它会先查已有章节再插入，幂等）
2. 或手动给表加 UNIQUE 约束：`CREATE UNIQUE INDEX IF NOT EXISTS idx_chapters_book_chapter ON chapters(book_id, chapter_num)`
3. 清理已有的重复行
