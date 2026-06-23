# RDK BSP 认知档案

## 概述
地瓜机器人 RDK 系列开发板（X5、S100、S600）的 BSP（Board Support Package）使用统一的构建框架 rdk-gen。
不同芯片使用不同的 BSP 版本，但构建流程高度一致。

## 芯片版本对照

| 芯片 | BSP版本 | Ubuntu | 内核 | 交叉编译工具链 | 源码来源 |
|------|---------|--------|------|----------------|----------|
| RDK X5 | 3.5.0 | 22.04 (jammy) | 6.1.112-rt43 | 13.2 (arm-gnu-toolchain) | GitHub D-Robotics/x5-manifest (repo sync) |
| RDK S100 | 4.0.5-Beta | 22.04 (jammy) | 6.1.158-rt58 | 11.3 (arm-gnu-toolchain) | 地平线内部发布 (tar.gz) |
| RDK S600 | 5.1.0 | 24.04 (noble) | 6.1.158-rt58 | 13.2 (arm-gnu-toolchain) | 地平线内部发布 (tar.gz) |

## BSP 源码目录结构（通用）

```
rdk-gen/
├── pack_image.sh             # 打包系统镜像入口
├── mk_kernel.sh              # 编译内核
├── mk_debs.sh                # 编译所有/单个 debian 包
├── download_deb_packages.sh  # 从官方服务器下载预编译 deb
├── download_samplefs.sh      # 下载 Ubuntu 根文件系统
├── hobot_customize_rootfs.sh # 定制 rootfs
├── build_params/             # 构建配置（soc型号、镜像版本、apt源等）
├── config/                   # 板级配置（vfat分区内容）
├── samplefs/                 # 根文件系统制作脚本
├── VERSION                   # BSP 版本号
└── source/                   # 源码目录（19个子模块）
    ├── kernel/               # Linux 内核源码
    ├── bootloader/           # uboot + miniboot
    ├── hobot-drivers/        # 地瓜自研驱动（摄像头ISP、BPU、IO等）
    ├── hobot-dnn/            # 深度学习推理框架（BPU）
    ├── hobot-camera/         # Sensor 驱动 + ISP tuning 参数
    ├── hobot-multimedia/     # 多媒体库（编解码、显示、音频）
    ├── hobot-multimedia-dev/ # 多媒体开发工具
    ├── hobot-multimedia-samples/ # 多媒体示例
    ├── hobot-spdev/          # 多媒体+推理融合框架
    ├── hobot-sp-samples/     # 融合框架示例
    ├── hobot-io/             # IO 库（GPIO/UART/SPI/I2C）
    ├── hobot-io-samples/     # IO 示例
    ├── hobot-utils/          # 系统工具（perf、cryptsetup等）
    ├── hobot-configs/        # 系统配置（启动项、服务）
    ├── hobot-wifi/           # WiFi 脚本
    ├── hobot-audio-config/   # 音频服务
    ├── hobot-firmware/       # 外设固件（网口/USB）
    ├── hobot-miniboot/       # 最小启动镜像 deb 包
    └── hobot-ethercat/       # EtherCAT 支持（S100/S600）
```

## 编译流程

正确顺序：
1. `pack_image.sh`（全量：下载samplefs + deb → 构建rootfs → 打包img）
   或 `pack_image.sh -p`（仅下载deb包和samplefs，不打包img）
   或 `pack_image.sh -l -p`（本地模式，不下载）
2. `mk_kernel.sh`（编译内核）
3. `mk_debs.sh`（编译所有hobot-* deb包）
4. `pack_image.sh -l`（本地打包镜像）

## 编译环境要求（Ubuntu 22.04）

依赖包见 README.md。关键依赖：
- 交叉编译工具链（见上方对照表）
- flex, bison, lz4（内核编译必需）
- qemu-user-static + binfmt-support（aarch64 仿真）
- python3 + pip (requirements.txt)
- fastecdsa（官方要求 <3.0.0，安装 3.0.1 可替代）

## 编译遇到的坑

1. **fastecdsa 版本冲突**：requirements.txt 要求 <3.0.0，pip 安装 3.0.1 预编译wheel可直接用
2. **lz4 工具缺失**：内核 CONFIG_KERNEL_LZ4=y 需要 lz4 命令行工具生成 Image.lz4
3. **intdeb-pkg 需要 Image.lz4**：需手动 lz4 -f Image Image.lz4 后重新 make intdeb-pkg
4. **S600 samplefs 404**：samplefs_desktop_noble-v0.0.6.tar.gz 在 archive.d-robotics.cc 上 404
5. **bindeb-pkg 失败**：因 LOCALVERSION 不一致导致 debian/rules 中重新链接 vmlinux 失败
6. **S100/S600 需内网访问**：samplefs 和 deb 包需要 archive.d-robotics.cc 内网源，外网编译受限
7. **repo init 卡死**：SSH key 权限问题，改用 https clone 手动拉取子模块
8. **X5 manifest path 解析**：manifest 中 path="source/xxx"，需额外处理避免 source/source 嵌套

## 当前本地状态（2026-06-23）

- /home/leaf-jammy/Develop/rdk-sdk/rdk-s600-bsp/ — 5.1.0，源码完整
- /home/leaf-jammy/Develop/rdk-sdk/rdk-s100-bsp/ — 4.0.5-Beta，源码完整
- /home/leaf-jammy/Develop/rdk-sdk/rdk-x5-bsp/ — 3.5.0，从 GitHub clone（41个子仓库）

## Downloads 打包文件

- rdk-s600-bsp_v5.1.0.tar.gz (1.8GB)
- rdk-s100-bsp_v4.0.5-Beta.tar.gz (1.7GB)
- rdk-x5-bsp.tar.gz (965MB)
