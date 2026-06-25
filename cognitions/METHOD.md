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

- **消息发送失败时，必须自动降级切换其他可用渠道**。例如 web_chat 失败时尝试 ros_msg，ros_msg 失败时尝试 email。不能因为一个渠道失败就丢弃消息。

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

## agent_loop pure-text reply 拦截机制 (2026-06-24)

之前 LLM 有时会绕过 message_send 直接生成纯文本 reply，用户看不到。

修复范围：
- `cs_core/src/agent_loop_node.cpp` 的 `single_step` 函数中，原先 `!has_tool_calls` 分支直接 return
- 现在改为自动注入一条 user 消息：
  ```
  messages_.push_back({{"role", "user"}, {"content", "reply无法被用户查看，请选择合适的消息渠道发送消息（message_send）"}});
  ```
- 效果：LLM 下一次推理时看到这条提醒，自动调用 message_send 重发

修复不涉及：
- `web_chat_server.py` — 只负责 SSE 推流和接收消息，不参与回复路由
- `cs_output` — 只负责工具执行，不参与决策

更改文件：
- `/home/leaf-jammy/Develop/Cloud-Soul/src/cs_core/src/agent_loop_node.cpp`
- 编译后需重启 cs_core 节点生效（由 Leaf 手动执行 stop_adam.sh / start_adam.sh）
- 备份在 `agent_loop_node.cpp.bak`

后续优化 (Leaf 修改):
- 提示词追加 `，如果没有事情需要通知，则不发`，避免空闲时每轮都发消息
- 提示词变为: `reply无法被用户查看，请选择合适的消息渠道发送消息（message_send），如果没有事情需要通知，则不发`

## message_send 渠道降级机制 - 待实现 (2026-06-24)

当前 message_send 每次只能指定一个 channel，如果该 channel 的 Action Server 超时或失败（返回 `动作服务器未就绪` / `工具执行超时`），消息直接丢失，没有降级逻辑。

问题现象：
- web_chat 渠道偶发 Action Server 未就绪 → 消息丢失
- 工具调用返回 `{"error":"工具执行超时"}` 或 `动作服务器未就绪`
- 没有自动切换到 ros_msg 或 email 降级发送

期望行为：
- channel 失败时自动尝试下一个可用渠道（web_chat → ros_msg → email）
- 或者 cs_output 的 message_send_node 内部支持多 channel 数组，依次尝试

代码位置：`cs_output/src/message_send_node.cpp`，每个 channel 独立处理 (`handle_email` / `handle_topic` / `handle_web_chat`)，无降级逻辑。

## RDK X5 板端原生编译内核 (2026-06-24)

在 RDK X5 开发板上用本地 gcc 原生编译 Linux 6.1.83 内核，全链路验证通过。

关键数据和步骤：
- **板端资源**: 8核 A55, 7GB RAM, 58GB eMMC（编译后占用 18GB/32%）
- **源码传输**: WSL2 → X5 通过 RNDIS USB 网卡 rsync，排除 .git/samplefs/deploy 后源 4.7GB→实际传 3.9GB，耗时约 2 分钟
- **编译时间**: ~50 分钟（make -j8 ARCH=arm64 Image modules），其中因 SSH 超时中断一次，续编后从 .o 增量继续
- **依赖**: bc 是内核编译必需品，X5 出厂镜像未预装，需 `apt install bc bison flex libssl-dev`
- **最终产物**: vmlinux 339MB, Image 23MB, 947 个 .ko 模块
- **安装**: `make modules_install` → 947 个模块安装到 /lib/modules/6.1.83 ；`cp arch/arm64/boot/Image /boot/Image`
- **重启验证**: 新内核正常启动，`uname -r` 显示 6.1.83，编译时间戳 2026-06-24 16:05:24

踩坑：
1. **SSH 超时 kill 了 make 进程** — 因 `make olddefconfig` 触发 60s 工具超时，SSH 被 agent_loop 的 shell_exec 超时机制 kill，连带 nohup 的 make 子进程也被终止。教训：长任务需要用 `nohup ... & disown` 或写 systemd timer / cron，确保父进程被 kill 后子进程存活。后续重启 make 时因为有 .o 文件可增量编译，损失不大。
2. **内核配置** — 板端没有 /boot/config 文件（是目录），通过 `zcat /proc/config.gz` 获取运行内核配置，6102 项全部继承。
3. **交叉工具链缺失** — WSL2 上没有 `aarch64-none-linux-gnu-` 工具链（mk_kernel.sh 需要 /opt/gcc-arm-11.2-2022.02-x86_64-...），但板端原生编译不需要交叉工具链，直接用本地 gcc 11.2.0 编译。

## nlohmann::json dump UTF-8 校验导致上下文压缩崩溃 (2026-06-25)

上下文压缩后，`save_context()` 和 LLM API 调用 (`body.dump()`) 抛出 `[json.exception.type_error.316] invalid UTF-8 byte at index 2000: 0x0A`，导致 agent_loop 进入死循环重试。

根因：nlohmann/json 3.10.5 的 `dump()` 默认校验字符串是否为合法 UTF-8。某条消息的 content 中包含不完整的多字节 UTF-8 序列（如 `E7 88 0A` — 两字节后遇到 0x0A 而非合法的 continuation byte 0x80-0xBF），触发校验失败。0x0A 本身合法，但在多字节序列中间出现就非法。

修复（commit 5dfd77c，develop 分支）：
- `agent_loop_node.cpp` 新增 `sanitize_utf8()` — 逐字节扫描，无效序列替换为 U+FFFD
- 新增 `sanitize_json_utf8()` — 递归清理 JSON 对象中所有 string 值
- `save_context()` catch 块：dump 失败后自动调用 sanitize + 重试
- `refresh_client()`：发送 LLM 前对每条消息做 UTF-8 清理

覆盖两条关键路径：上下文文件保存 + LLM API 请求构造。编译后需重启 Adam 生效。

教训：
- nlohmann::json 的 dump() 不是纯序列化，它会做 UTF-8 校验
- 任何可能包含外部数据（LLM 输出、文件读取、shell 输出）的字符串，在存入 nlohmann::json 前应考虑 sanitize
- 防御性编程：关键路径的 dump() 调用应包裹 try-catch + sanitize + retry