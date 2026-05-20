## 错误处理与恢复

### Target 丢失 / createTarget 被拦截

当 `Target.createTarget` 创建的 target 在几秒后消失（Edge 弹窗拦截器）：
1. **不要重试 createTarget**——改用 `Page.navigate` 或 `window.location.href` 在已有 zhipin target 上导航
2. 用 `Target.getTargets()` 找到已有的 zhipin.com page target
3. 用 CDP `Page.navigate` + `Runtime.evaluate('window.location.href=URL')` 组合导航

### 变量重复声明错误

在同一个 target 上连续 `Runtime.evaluate` 时，`const`/`let` 声明的变量会持久化：
- ❌ `const chatItems = ...` → 第二次执行报 `SyntaxError: Identifier already declared`
- ✅ 用 `var` 或不声明，或用 IIFE 包裹：`(() => { const x = ...; return x; })()`
- ✅ 最简单：直接返回表达式，不用变量

### CDP 连接不稳定

- 单次会话发送 3-4 条后 CDP 可能不稳定（target 丢失、evaluate 失败）
- 如果连续 2 条失败，停止并报告用户
- Hermes 内置 browser（`browser_navigate`）不共享 Edge 登录态，点击"立即沟通"会跳转到首页而非打开聊天
- **BOSS 直聘必须走 CDP 连 Edge**，不能用 Hermes 内置 browser

### SPA 页面导航

- `Page.navigate` 在 BOSS SPA 上可能不生效（页面标题不变）
- **可靠方式**：`Runtime.evaluate('window.location.href = URL')` 强制导航
- 导航后等 **6 秒**（SPA 渲染慢，3 秒不够）

## PITFALLS

- **选择器随场景变化**：详情页弹窗用 `textarea.input-area` + `div.send-message`；聊天页面用 `div.chat-input` + `.btn-send`。必须先判断场景
- **contenteditable div 用 innerText 不用 value**：聊天页面的 `.chat-input` 是 div，`.value` 为 undefined
- **BOSS 前端用 React/Vue**：直接设 DOM 值不触发状态更新，必须手动 dispatch `input` + `change` 事件
- **"继续沟通"按钮不弹窗**：如果之前已开聊，点击"继续沟通"会跳转到 `/web/geek/chat` 而非弹出聊天框
- **Target.createTarget 被 Edge 拦截**：新标签被弹窗拦截器吞掉，targetId 创建后消失。改用已有 target + `window.location.href`
- **`const` 变量跨 evaluate 持久化**：同一 target 的 JS context 共享，重复 `const` 声明会 SyntaxError
- 如果 detail 页面被重定向到登录页，说明 cookie 失效，需要用户重新登录
- BOSS 可能弹出"完善简历"等中间弹窗拦截，点击"立即沟通"后应检查是否有遮罩层
- **聊天页 DOM 可能显示空列表但消息实际已送达**：场景 B 跳转到 `/web/geek/chat` 后，`.message-list .message-item` 可能为空（count=0），但用户在 Edge 界面上能看到消息已送达。不要仅凭 DOM 空就判定发送失败——如果 `btn-send` 已 click 且无报错，大概率成功了
- **"继续沟通"不一定自动跳转聊天页**：点击"继续沟通"后有时页面 URL 不变（还在详情页），此时可手动 `window.location.href = 'https://www.zhipin.com/web/geek/chat'` 导航过去
- **Edge 重启后需重新导航**：`pkill Edge → open --args --remote-debugging-port=9222` 后，target 是新标签页（edge://newtab），需要先导航到 zhipin.com 才能操作。等 5-8 秒确认 CDP 就绪
