# RDK 手册文档测评报告

**来源**: https://github.com/D-Robotics/rdk_doc | **文件数**: 323 .md | **总行数**: ~103K

---

## 文档结构

| 章节 | 文件数 | 行数 |
|------|--------|------|
| 01 快速开始 | 27 | 5,072 |
| 02 系统配置 | 5 | 1,458 |
| 03 基础应用开发 | 99 | 19,386 |
| 04 算法应用开发 | 5 | 893 |
| 05 机器人开发 | 66 | 18,932 |
| 06 应用案例 | 2 | 1,176 |
| 07 进阶开发 | 82 | 49,237 |
| 08 常见问题 | 7 | 4,659 |
| 09 附录 | 28 | 2,819 |
| 10 版本发布 | - | - |

---

## 🔴 发现的问题

### P1 - "地平线" 品牌残留 (7处)
公司已更名为"地瓜机器人/D-Robotics"，以下文件仍有"地平线"：
- `05_Robot_development/03_boxs/spatial/hobot_vio.md`
- `05_Robot_development/03_boxs/spatial/stereo_imu_cam.md`
- `05_Robot_development/03_boxs/detection/hobot_dosod.md`
- `05_Robot_development/03_boxs/driver/hobot_bev.md`
- `05_Robot_development/03_boxs/driver/hobot_centerpoint.md`

### P2 - 过期 CDN 链接 (172处)
`rdk-doc.oss-cn-beijing.aliyuncs.com` 大量出现。虽然图片当前可访问，但 OSS 域名未来可能变更。建议统一为 CDN。

### P3 - 旧文档中心 URL (10处)
`developer.d-robotics.cc/rdk_doc` 已迁移至 `/rdk_doc_center`，手册仍有10处旧链接。

### P4 - 硬编码默认 IP `192.168.127.10` (24处)
有线以太网默认 IP。实际闪连 USB 是 `192.168.128.10`，WiFi 是 DHCP。不同网络模式 IP 差异没有突出说明。

### P5 - arm64 vs aarch64 术语不一致
手册用 `arm64`(Debian习惯)，内核用 `aarch64`(ARM官方)。新用户可能困惑。

### P6 - 闪连文档不够突出
- 硬件介绍中"闪连接口 (USB Type C)"是序号3，与供电接口外观相同
- `remote_login.md` 中闪连 IP 表仅 X5 镜像 3.0.0
- 缺少完整的闪连使用指南（Windows驱动安装、RNDIS配置等）

### P7 - ROS2 apt 源可能失效
多处文档指导添加 `packages.ros.org` 源。实测该源返回 400 Bad Request。FAQ Q10 提到了但解决方案仅建议检查网络和时间。

### P8 - GPIO 文档 Python 版本过时
`gpio.md` 示例显示 `Python 3.8.10`，当前系统是 `Python 3.10.12`。

### P9 - 缺少关键故障排查文档
以下常见问题手册中无说明：
- ❌ systemd-resolved DNS 劫持 / WiFi captive portal
- ❌ 系统时钟同步 (NTP)
- ❌ 闪连 USB 网络在 Windows 上的驱动安装
- ❌ WiFi 连接企业网络 (802.1X) 配置
- ❌ `/opt/ros` vs `/opt/tros` 双 ROS2 版本选择指南

### P10 - 烧录文档镜像版本号过时
`01_system_burn.md` 引用 `3.3.3-arm64.img`，当前最新为 3.5.0。

### P11 - 废弃 API 链接
`RDK.md` 引用旧 API 地址 (v1 API)，可能已过期：
- `developer.d-robotics.cc/api/v1/fileData/documents_pi/index.html`
- `developer.d-robotics.cc/api/v1/static/fileData/...`

### P12 - 获取系统信息方式不一致
多处用 `cat /etc/version`，2.1.0+ 推荐 `rdkos_info`，输出格式不同。

### P13 - Ubuntu 20.04 残留 (202处)
手册中大量提及 Ubuntu 20.04/Foxy。当前主力 3.x 系统均为 22.04/Humble，旧版内容应归档而非混在主文档。

### P14 - hello_world 用例依赖 apt 安装
`hello_world.md` 指导 `apt install ros-humble-examples-*`。但 apt 源可能不可用（P7），推荐改为板卡已有 demo 替代。

---

## ✅ 文档做得好的方面

- GitHub 开源，方便社区贡献
- Docusaurus 框架，中英文、多版本切换
- 章节结构清晰
- 大量 Mermaid 流程图
- FAQ 覆盖常见问题
- 每个 demo 有 README
- TROS/ROS FAQ 写得详细（Q2 解释 TROS vs ROS2 关系）

---

## 改进建议

1. 批量替换：地平线→地瓜机器人, 旧URL→新URL
2. 新增：闪连完整指南
3. 新增：网络故障排查（DNS劫持、captive portal）
4. 新增：系统时间同步
5. 更新：所有镜像版本号示例到 3.5.0
6. 增加：/opt/ros vs /opt/tros 对比选择说明
7. 增加：快速诊断脚本
8. 归档：Ubuntu 20.04/Foxy 内容到独立章节
