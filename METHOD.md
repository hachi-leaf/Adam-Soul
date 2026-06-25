
## agent_loop pure-text 拦截死循环 (2026-06-25 00:13)

空闲时 Adam 输出纯文本（如"静默"），C++ `single_step` 检测 `!has_tool_calls` 后自动注入提醒。LLM 重新输出纯文本 → 再次拦截 → 无限循环。每轮 ~13s，2 小时累计 ~550 轮。

**临时打破**：输出差异化内容让 LLM 调用 message_send。

**根因**：拦截逻辑无去重/频率限制。

**修复方向**（待 Leaf 实现）：
- 连续 N 轮纯文本内容相同时放行不注入
- 或每分钟最多注入 1 次提醒

**关联文件**：`cs_core/src/agent_loop_node.cpp` → `single_step` 函数。
