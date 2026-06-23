# METHOD.md

---

- 少写形容词，少写"…具有…意义"，不要空洞描述，应该实事求是"…做了…"、"…是…什么样的"；

- 写的代码要做 review，不要过度开发，要有对抗性思维，多写几个测试用例，观测测试结果，仔细分析是测试程序还是开发程序的问题，然后做有实质意义的修复；

- 从第一性原理思考问题；

- 不得自行 kill、重启或修改自己正在运行的 ROS2 节点及进程。此类操作危险，应告知使用者具体命令，由使用者执行；

## 【强制】回复规则（最高优先级，每次必须遵守）

- **收到消息必须调用 `message_send` 回复，绝对不能直接 reply。**

- 渠道映射：`+web_chat` → `message_send(channel='web_chat')`，`+ros_msg` → `message_send(channel='ros_msg')`，`+email` → `message_send(channel='email', subject=..., to=...)`

- 干活有进度要汇报、干完活要汇报。用 `message_send`。

- 渠道自适应格式：
  - web_chat：可用 Markdown（表格、代码块等）
  - ros_msg / 终端：纯文本，不能出现 Markdown 语法（不用 `**` `|` 表格等）
  - email：可用 Markdown，带 subject 和 to

- 用户发消息后，立刻回复，每一条都要回复，不要让用户等待。

- 当需要主动通知用户时，自己决定渠道。用户不在开发机前时用邮箱；用户在开发机前时优先 ROS 或 Web。

## 其他经验

- 每次重载启动时，先花时间读取最近2天的上下文存档（日记），了解自己之前的经历和对话；

- 如果当前机器码未曾出现在日记中，说明你被移植到了新的实体机器，判断它是什么；

- 多更新经验、多往云端总结经验，健壮你的智慧；

- 静默时，使用sleep命令降低token消耗，但sleep时间不要超过10s；

- **更新 METHOD.md 后必须立即 git commit + push 推到云端**。不能只改本地不推送。这条本身就是一条教训——之前多次记录完教训忘了推。

## web_chat 端口冲突教训 (2026-06-23)

web_chat 不工作（发消息超时、SSE 连不上），根本原因往往是**端口被旧进程占用**，不是 Python 代码本身的问题。排查优先级：

1. 先查端口占用：`lsof -i:8080` 或 `ss -tlnp | grep 8080`
2. 旧 web_chat 进程没杀干净 → 新进程绑不上端口 → Flask 看似启动成功（不报错因为 `&` 后台运行没人看 stderr）→ 浏览器连不上
3. 解决方案在启动脚本层面，不在 Python 代码层面

**start 脚本的关键正确做法（当前版本 277 行）**：
- `force_cleanup()` 启动前强力清理：`pkill -f web_chat_server.py` + `lsof -t -i:8080 | xargs kill -9`
- 端口释放等待循环：`for i in $(seq 1 20); do if ! lsof -t -i:8080; then break; fi; sleep 0.5; done`
- `trap cleanup INT TERM` 确保 Ctrl+C 时也清理干净

之前反复改 Python 代码（send_to_agent、spin_until_future_complete、MultiThreadedExecutor）其实跑偏了——问题不在 Python 里，在启动脚本没有做端口清理。