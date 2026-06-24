# 这个世界

有关你认识的人、认识的地点、认识的物品、发生的事情等世界的信息，都在这：

---

- 开发框架: Cloud-Soul (https://github.com/hachi-leaf/Cloud-Soul)，基于 ROS2 的 AI Agent 运行时
- 记忆仓库: Adam-Soul (https://github.com/hachi-leaf/Adam-Soul)，Git 托管云端记忆
- 语言: 中文为主，英文为辅

## 开发环境

- 主力开发机: LUOBO-4RDM0SB (Intel i5-1340P, 16GB RAM, 1TB SSD, Windows + WSL2 Ubuntu 22.04)
- 用户: leaf-jammy (旧实例 7ab95b98 已废弃，新实例 3e760dd3 于 2026-06-23 启用)
- ROS2 Humble
- BSP 源码路径: /home/leaf-jammy/Develop/rdk-sdk/

## 开发板

### RDK X5
- SoC: D-Robotics X5
- IP: 192.168.128.10 (USB RNDIS)
- BSP: 3.5.0, Ubuntu 22.04 (jammy), 内核 6.1.112-rt43
- 交叉编译: arm-gnu-toolchain 13.2
- 源码: GitHub D-Robotics/x5-manifest (repo sync)
- 识别串: `D-Robotics RDK X5 V1.0`
- 特点: 8核 A55, BPU 独立供电域, 40PIN GPIO, CAN FD

### RDK S100
- BSP: 4.0.5-Beta, Ubuntu 22.04 (jammy), 内核 6.1.158-rt58
- 交叉编译: arm-gnu-toolchain 11.3
- 源码: 地平线内部发布 (tar.gz)
- 有 EtherCAT 支持
- 本地路径: /home/leaf-jammy/Develop/rdk-sdk/rdk-s100-bsp/

### RDK S600
- BSP: 5.1.0, Ubuntu 24.04 (noble), 内核 6.1.158-rt58
- 交叉编译: arm-gnu-toolchain 13.2
- 源码: 地平线内部发布 (tar.gz)
- 有 EtherCAT 支持
- 本地路径: /home/leaf-jammy/Develop/rdk-sdk/rdk-s600-bsp/
- samplefs desktop noble 在 archive.d-robotics.cc 上 404

## BSP 编译要点

- 构建框架: rdk-gen，三套 BSP 流程一致
- source/ 下 19 个子模块: kernel, bootloader, hobot-drivers, hobot-dnn, hobot-camera, hobot-multimedia, hobot-io, hobot-configs 等
- 编译入口: pack_image.sh / mk_kernel.sh / mk_debs.sh
- 依赖: flex, bison, lz4, qemu-user-static, binfmt-support, python3+pip
- 关键坑:
  - fastecdsa 版本要求 <3.0.0，实际 3.0.1 wheel 可用
  - lz4 工具缺失会导致内核编译失败
  - S100/S600 需要 archive.d-robotics.cc 内网源
  - X5 manifest path 解析可能产生 source/source 嵌套
  - S600 samplefs 404 问题未解决

## BSP 打包备份

- rdk-x5-bsp.tar.gz (965MB)
- rdk-s100-bsp_v4.0.5-Beta.tar.gz (1.7GB)
- rdk-s600-bsp_v5.1.0.tar.gz (1.8GB)
