## agent_loop pure-text 拦截死循环 (2026-06-25 00:13)

空闲时 Adam 输出纯文本（如"静默"），C++ `single_step` 检测 `!has_tool_calls` 后自动注入提醒。LLM 重新输出纯文本 → 再次拦截 → 无限循环。每轮 ~13s，2 小时累计 ~550 轮。

**临时打破**：输出差异化内容让 LLM 调用 message_send。

**根因**：拦截逻辑无去重/频率限制。

**修复方向**（待 Leaf 实现）：
- 连续 N 轮纯文本内容相同时放行不注入
- 或每分钟最多注入 1 次提醒

**关联文件**：`cs_core/src/agent_loop_node.cpp` → `single_step` 函数。

## 思考模式 + reasoning_content 规则 (2026-06-25)

**强制规则**：
- `client.set_thinking_enabled(true)` 必须在每次创建 OpenAIClient 后调用
- `refresh_client` 中 **禁止** `clean.erase("reasoning_content")` —— 思考内容必须保留在消息列表中
- `single_step` 中 assistant_msg 必须携带 `reasoning_content`（如果 reply 中有的话）
- 思考模式不能被任何代码修改关闭，这是 Leaf 的核心需求

**关联文件**：`cs_core/src/agent_loop_node.cpp` 第 654 行附近、第 485/515-516 行。

## 上下文压缩双 RULE + 空对象问题 (2026-06-25)

**现象**：压缩后上下文文件出现双份 RULE 系统提示词（index 0 和 index 2 各一条）和空对象 `{}`（role 为空），导致 LLM 调用失败。

**根因**：
- `compress_context` 的 `for (auto& m : recent)` 循环中，`recent` 可能包含 system 消息，push 后与前面的 system 消息重复
- `refresh_client` 的 `add_message` 要求消息必须含 `role`，空对象会触发 `throw`

**修复**：
- `compress_context`：`for (auto& m : recent)` 中加 `if (m.value("role", "") == "system") continue;`
- `refresh_client`：`for (const auto& m : messages_)` 中加 `if (m.empty() || !m.contains("role")) continue;`

**关联文件**：`cs_core/src/agent_loop_node.cpp` → `compress_context` 和 `refresh_client` 函数。

## 修改 agent_loop_node.cpp 后必须 colcon build + 重启 (2026-06-25)

`agent_loop_node.cpp` 是 C++ 编译型代码，修改后：
1. `colcon build --packages-select cs_core`
2. 重启 Adam（`stop_adam.sh` + `start_adam.sh`）
仅改源码不编译不重启 = 什么都没改。