# RDK X5 板卡实测报告

**测试时间**: 2026-06-23 | **系统版本**: 3.5.0 | **板卡**: RDK X5 V1.0

---

## 1. 系统概览

| 项目 | 值 |
|------|-----|
| 主机名 | ubuntu |
| 内核 | 6.1.83 aarch64 |
| OS | Ubuntu 22.04.5 LTS (Jammy) |
| CPU | 8核 ARM A55, 1.2GHz (当前频率) |
| 内存 | 6.9GiB (可用6.6GiB) |
| 磁盘 | 58G 总(已用 8.4G) |
| 温度 | ~65.5°C |
| Python | 3.10.12 |
| GCC | 11.2.0 |
| CMake | 3.22.1 |

## 2. 软件环境

### ROS2 双版本共存
- `/opt/ros/humble` - 标准 ROS2 Humble
- `/opt/tros/humble` - 地瓜魔改 TogetheROS.Bot (18个 hobot_* 包)
- 两者 API 兼容，但不能同时 source

### 核心软件包
| 包 | 版本 | 功能 |
|----|------|------|
| hobot-dnn | 3.0.4 | BPU 推理框架 |
| hobot-cv | - | 图像处理加速 (裁剪/缩放/旋转/高斯模糊) |
| hobot-camera | 3.1.1 | 摄像头传感器支持 |
| hobot-io | 3.1.4 | IO 外设支持 |
| hobot-llamacpp | 0.3.0 | 大模型推理 (依赖摄像头输入) |
| hobot-yolo-world | - | YOLO-World 检测 |
| hobot-falldown | - | 跌倒检测 |
| body-tracking | - | 人体追踪 |
| gesture-control | - | 手势识别 |
| audio-tracking | - | 音频追踪 |
| Hobot.GPIO | 0.0.2 | Python GPIO 库 |

### apt 源
- `http://archive.d-robotics.cc/ubuntu-rdk-x5 jammy main`
- `http://packages.ros.org/ros2/ubuntu jammy main` (⚠️ 返回400)
- `http://ports.ubuntu.com jammy*`

## 3. 网络连接

| 接口 | IP | 状态 |
|------|-----|------|
| usb0 (闪连) | 192.168.128.10/24 | UP |
| wlan0 (WiFi) | 10.0.0.42/24 | UP, SSID: D-robotics-sz, -39dBm |
| eth0 | 无 | DOWN |

## 4. 实测 Demo 结果

| Demo | 状态 | 备注 |
|------|------|------|
| ResNet18 分类 | ✅ 通过 | BPU 推理正常, NV12 输入 |
| MobileNetV2 分类 | ✅ 通过 | 同上 |
| YOLOv5s MIPI | ❌ 无摄像头 | i2c 探测传感器失败 |
| USB Camera YOLOv5x | ❌ 无摄像头 | "No USB camera found" |
| GPIO | ✅ 可用 | Hobot.GPIO 导入正常 |
| ROS2 daemon | ✅ 运行中 | 基础 topics 正常 |
| hobot_cv | ⚠️ 需要输入 | resize_example 无图崩溃 |
| hobot_llamacpp | ⚠️ 未测试 | 需要摄像头输入 |

## 5. TROS 功能包可执行文件

18 个 hobot_* 包，66 个 launch 文件：
- 人脸检测/关键点、跌倒检测、手势识别、人体追踪
- 音频追踪、立体视觉、MOT、YOLO-World
- 目标检测 (hobot_dosod)、图像处理加速 (hobot_cv)
- LLM 推理 (hobot_llamacpp)、编解码 (hobot_codec)

---

## 🔴 发现的问题

### P1 - 系统时钟错误 (严重)
- 初始时钟为 Unix epoch (1970)
- apt update 报 "not valid yet (invalid for another 9624d)"
- 手动 `date -s` 从 HTTP Date 头同步后修复
- 写入硬件时钟后重启可保持

### P2 - DNS 被 Captive Portal 劫持 (严重)
- WiFi "D-robotics-sz" 的 DNS 被路由器劫持
- baidu.com → 198.18.10.134 (captive portal)
- 即使指定 8.8.8.8 DNS, host 依然返回错误地址
- 路由器在拦截 DNS 请求
- 绕过: /etc/hosts 手动映射, 或直连 IP

### P3 - ROS2 apt 源失效 (中等)
- http://packages.ros.org/ros2/ubuntu → 400 Bad Request
- 当前已预装 ROS2, 影响较小
- 但新增安装功能包会受阻

### P4 - 无摄像头 (功能限制)
- /dev/video* 不存在
- 未连接 MIPI/USB 摄像头
- 大量 demo 无法测试 (检测/分割/USB摄像头/web显示)

### P5 - chattr +i 锁定 resolv.conf (操作失误)
- 之前操作 `chattr +i /etc/resolv.conf` 未恢复
- 后续写入报 "Operation not permitted"
- 需 `chattr -i` 解锁

### P6 - 温度偏高
- 空闲温度 ~65.5°C
- 接近降频阈值 95°C
- 建议加散热措施

---

## ✅ 好的方面

- 预装完整的 TROS + ROS2, 含18个功能包
- /app/pydev_demo/ 下有9大类丰富的 demo
- BPU 推理秒级出结果
- GPIO 库易用 (4种编码模式)
- 闪连 USB + WiFi 双网络就绪
- 文档 GitHub 开源

## 建议

1. 增加默认 NTP 自动校时
2. 闪连/网络诊断工具内置于系统
3. 出厂自带简单诊断脚本 (一键测 BPU/网络/camera)
4. 建议搭配摄像头+散热片套餐

## 6. BPU 性能基准

**测试方法**: `hbm_runtime.HB_HBMRuntime.run()` 接口, 50-100 次推理取平均, 不含前后处理。

| 模型 | 输入尺寸 | FPS | 延迟 |
|------|----------|-----|------|
| MobileNetV2 | 224×224 | 590.9 | 1.7ms |
| EfficientNet-Lite0 | 224×224 | 547.5 | 1.8ms |
| ResNet18 | 224×224 | 325.7 | 3.1ms |
| YOLOv8 | 640×640 | 101.5 | 9.9ms |
| YOLOv5s | 672×672 | 41.6 | 24.0ms |
| YOLOv5x | 672×672 | 10.0 | 100.1ms |

**预装模型**: 34 个 .bin 文件, 涵盖分类/检测/分割/姿态估计/CenterNet/FCOS/DeepLabV3+ 等。

## 7. 存储性能

| 操作 | 速度 |
|------|------|
| SD写 | 284 MB/s |
| SD读 | 1.4 GB/s |

## 8. 外设接口

| 接口 | 状态 |
|------|------|
| USB 闪连 | ✅ usb0 192.168.128.10 |
| WiFi | ✅ wlan0 10.0.0.42, -39dBm |
| 以太网 | ⚠️ DOWN (无连接) |
| CAN FD | ⚠️ DOWN (未配置) |
| HDMI | ❌ 无显示器连接 |
| 音频 | ✅ ES8326 I2S 编解码器 (record+playback) |
| MIPI CSI | ⚠️ 传感器探测失败 (无摄像头) |
| 40PIN GPIO | ✅ Hobot.GPIO 库就绪 |
| SD卡 | ✅ 读1.4GB/s 写284MB/s |
| CPU调速器 | schedutil (300MHz-1.5GHz) |

## 9. 多媒体支持

- **传感器兼容**: sc230ai, sc1330t, irs2875, f37, imx415, imx477, ar0233, os08c10, sc231ai, gc2053 等
- **GDC 预处理**: 多 sensor 的 GDC bin 文件预装
- **双摄像头**: 2路 MIPI CSI 4-lane
- **ISP/VIN/VSE**: 支持 V4L2 接口访问
- **音频**: ES8326 编解码器, 支持录音+播放
- **GPU**: 2D/3D 示例就绪

## 10. 已确认可工作的 Demo

| Demo | API | 输入 | 状态 |
|------|-----|------|------|
| ResNet18 分类 | hbm_runtime.run() | NV12 224x224 | ✅ |
| MobileNetV2 分类 | hbm_runtime.run() | NV12 224x224 | ✅ |
| YOLOv5s 检测 | hbm_runtime.run() | NV12 672x672 | ✅(仅推理) |
| YOLOv8 检测 | hbm_runtime.run() | NV12 640x640 | ✅(仅推理) |
| GPIO | Hobot.GPIO | - | ✅ |

## 11. 受限无法测试的 Demo

这些需要摄像头/显示器/电机等外设：
- MIPI 摄像头采集 + 实时检测 (YOLOv5s/v8/v10/v11)
- USB 摄像头采集
- Web 显示 (nginx + websocket)
- 人脸检测/关键点/跌倒检测/手势识别
- 人体追踪/音频追踪
- 立体视觉/深度估计
- LLM 推理 (hobot_llamacpp, 依赖摄像头)
- YOLO-World 开放词汇检测
- 语音识别/TTS (hobot_audio)
- AMR 导航
- 深度学习巡线
