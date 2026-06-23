# RDK X5 测评报告

**测试日期**: 2026-06-23 | **测试者**: Adam (Cloud-Soul AI Agent)  
**板卡**: D-Robotics RDK X5 V1.0 | **系统**: 3.5.0 | **连接**: USB 闪连 + WiFi

---

## 报告索引

| 文件 | 内容 |
|------|------|
| [01_board_review.md](./01_board_review.md) | 板卡硬件/软件/网络/BPU/demo 实测 |
| [02_manual_review.md](./02_manual_review.md) | 手册文档 323 个 md 文件逐章审查 |

## 总问题数

- 板卡实测: **6** 个问题 (2严重/1中等/2功能限制/1操作失误)
- 手册文档: **14** 个问题 (品牌残留/过期链接/硬编码/缺失文档/版本过时)

## 快速摘要

### 板卡 ✅
- BPU 推理正常 (ResNet18/MobileNetV2 秒级)
- TROS 功能完整 (18个功能包, 66个 launch 文件)
- 闪连 USB + WiFi 双网络就绪
- GPIO 库可用

### 板卡 ❌
- 系统时钟初始错误 (1970年)
- WiFi DNS 被 captive portal 劫持
- ROS2 apt 源 400 错误
- 无摄像头 (demo 受限)
- 空闲温度 ~65°C

### 手册 ❌
- "地平线"品牌残留 7处
- 过期OSS链接 172处
- 硬编码IP `192.168.127.10` 24处
- 缺少闪连/DNS故障排查/NTP配置文档
- 镜像版本号过时 (3.3.3 → 应为 3.5.0)
- Ubuntu 20.04/Foxy 残留 202处

---

## 使用方式

本报告可直接 open PR 到 [D-Robotics/rdk_doc](https://github.com/D-Robotics/rdk_doc)
