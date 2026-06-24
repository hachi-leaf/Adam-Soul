# WORLD

---

- 开发框架: Cloud-Soul (https://github.com/hachi-leaf/Cloud-Soul)，基于 ROS2 的 AI Agent 运行时
- 记忆仓库: Adam-Soul (https://github.com/hachi-leaf/Adam-Soul)，Git 托管云端记忆
- 语言: 中文为主，英文为辅

## 开发环境

- 主力开发机: LUOBO-4RDM0SB (Intel i5-1340P, 16GB RAM, 1TB SSD, Windows + WSL2 Ubuntu 22.04)
- 用户: leaf-jammy (旧 WSL2 实例 7ab95b98 已废弃，新实例 3e760dd3 于 2026-06-23 启用)
- ROS2 Humble
- BSP 源码路径: /home/leaf-jammy/Develop/rdk-sdk/

## 开发板

### RDK X5
- SoC: D-Robotics X5
- IP: 192.168.128.10 (USB RNDIS)
- 识别串: `D-Robotics RDK X5 V1.0`
- BSP: 3.5.0, Ubuntu 22.04 (jammy), 内核 6.1.112-rt43
- 交叉编译: arm-gnu-toolchain 13.2
- 源码: GitHub D-Robotics/x5-manifest (repo sync)
- 特点: 8核 A55 schedutil, BPU hpu3501 独立供电域, 40PIN GPIO, CAN FD, 双温控 zone (CPU/DDR)
- 板载多媒体: GPU 2D/3D OpenCL, Codec/OSD/Audio 编译 OK, 30 种摄像头传感器驱动

### RDK S100
- BSP: 4.0.5-Beta, Ubuntu 22.04 (jammy), 内核 6.1.158-rt58
- 交叉编译: arm-gnu-toolchain 11.3
- 源码: 地平线内部发布 (tar.gz)
- 有 EtherCAT 支持

### RDK S600
- BSP: 5.1.0, Ubuntu 24.04 (noble), 内核 6.1.158-rt58
- 交叉编译: arm-gnu-toolchain 13.2
- 源码: 地平线内部发布 (tar.gz)
- 有 EtherCAT 支持
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