## 安全规则

- **每条消息间隔 ≥ 10 秒**，避免触发反爬/限流
- **发送前必须从 pitch md 文件重新读取待发列表**（source of truth 原则，不能凭上下文记忆硬发）
- **发送前确认话术内容**，用户可能需要调整
- **CDP 不稳定时停止发送**：连续 2 条失败就停，报告用户

## PITFALLS

### 🚨 异常诊断

| 症状 | 判断 | 修复 |
|------|------|------|
| 弹窗没出现（hasTextarea=false） | 未登录或被限流 | 停止并报告用户 |
| 发送按钮点不动 | textarea value 未被框架识别 | 重新 focus + dispatch input/change 事件 |
| Target 丢失 / evaluate 连续失败 | CDP session 不稳定 | 停止并报告用户（参考 boss-zhipin-cdp 的重启流程） |
| detail 页被重定向到登录页 | cookie 失效 | 用户需重新扫码登录 |
| 聊天页 DOM 显示空列表但消息实际已送达 | `.message-list .message-item` 可能为空 | 如果 btn-send 已 click 且无报错，大概率成功，不要判定为失败 |

### 选择器与 DOM 坑

- **选择器随场景变化**：场景 A 用 `textarea.input-area` + `div.send-message`；场景 B 用 `div.chat-input` + `.btn-send`。必须先判断场景
- **contenteditable div 用 innerText 不用 value**：`.chat-input` 是 div，`.value` 为 undefined
- **必须 dispatch input + change 事件**：BOSS 用 React/Vue，直接设 DOM 值不触发状态更新
- **"继续沟通"不一定自动跳转**：有时页面 URL 不变，需手动 `window.location.href = 'https://www.zhipin.com/web/geek/chat'`
- BOSS 可能弹出"完善简历"等中间弹窗拦截，点击后检查是否有遮罩层

### 导航与 target

- **不要用 Target.createTarget**：Edge 弹窗拦截器会吞掉。用已有 target + `window.location.href`
- **const 变量跨 evaluate 持久化**：同一 target JS context 共享，重复 const 声明会 SyntaxError → 用 IIFE
- **Edge 重启后需重新导航**：重启后 target 是 edge://newtab，需先导航到 zhipin.com，等 5-8 秒确认就绪

更多通用 CDP 异常处理见 **boss-zhipin-cdp** skill 的 PITFALLS 章节。