# 场景 B：CDP target 丢失（boss-send-message）

## 错误描述

我在用 CDP 给 BOSS 直聘发投递话术。已经成功发了 2 条。

发第 3 条时遇到以下情况：
1. 用 `window.location.href` 导航到了新的 JD 详情页
2. 等 6 秒后执行 Runtime.evaluate 检查按钮状态
3. Runtime.evaluate 返回错误："No target with given id found"
4. 用 Target.getTargets 检查，发现之前一直用的 target 还在列表里

请帮我诊断原因并给出修复方案。

## 期望的诊断

根因：CDP session 在连续操作 3-4 次后可能不稳定。虽然 Target.getTargets 显示 target 还在，但 Runtime.evaluate 的 session 已经失效。

期望修复：
1. 连续 2 条失败应停止并报告用户
2. 尝试在同一个 target 上重新 attach（重新获取 session）
3. 如果仍然失败，可能需要重启 Edge 或重新建立 CDP 连接
