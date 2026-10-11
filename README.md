# Horizon HZ51 OpenWrt

Horizon HZ51 AX5400 5G CPE 的 OpenWrt 固件组件、LuCI 插件与局域网测速页面。

> 本项目来自工程样机上的实际适配。固件文件是原始 NAND/UBI 组件，不是 LuCI `sysupgrade` 安装包。刷写前请先备份本机 ART、BOOTCONFIG、APPSBLENV、`usr_config` 和 `usr_factory`，错误写入可能导致设备变砖。

## 硬件配置

以下规格根据 HZ51 参数图整理：

| 项目 | 参数 |
|---|---|
| 型号 | Horizon HZ51 |
| 5G / LTE 标准 | 5G SA/NSA Sub-6，3GPP Release 16，LTE Cat 19 |
| 5G NSA 速率 | 下行最高 3.2 Gbps，上行最高 600 Mbps |
| 5G SA 速率 | 下行最高 2.4 Gbps，上行最高 900 Mbps |
| LTE Cat 19 速率 | 下行最高 1.6 Gbps，上行最高 200 Mbps |
| 支持网络 | 5G SA、5G NSA、FDD-LTE、TDD-LTE |
| 5G NR 频段 | n2/n5/n7/n12/n13/n14/n25/n26/n29/n30/n38/n41/n48/n66/n70/n71/n77/n78 |
| LTE FDD 频段 | B2/B4/B5/B7/B12/B13/B14/B17/B25/B26/B29/B30/B66/B71 |
| LTE TDD 频段 | B38/B41/B42/B43/B48 |
| 5G 基带 | Qualcomm SDX62，4×4 MIMO |
| Wi-Fi | Wi-Fi 6，802.11a/b/g/n/ac/ax，4×4 MIMO，标称最高 4804 Mbps，最多 128 个客户端 |
| 参数图标注的 Wi-Fi 速度 | 最高 866 Mbps |
| 载波聚合 | 下行最高 5CA，上行最高 2FR1 CA |
| 语音 | VoIP / VoLTE |
| 网口 | 1× 最高 2.5 Gbps WAN，1× 最高 1 Gbps LAN（当前 OpenWrt 可将两口配置为 LAN） |
| SIM | 4FF Nano SIM |
| 天线 | 4× 4G/5G 内置天线，6× Wi-Fi MU-MIMO 天线 |
| 原厂管理 | 本地 Web GUI、Horizon DMS、TR-069 远程升级 |
| 认证 | FCC |

实机系统信息：OpenWrt Chaos Calmer 15.05.1、`ipq/ipq50xx`、Linux 4.4.60、ARMv7、约 128 MiB NAND。

## LuCI 插件

### RGB LED Control

文件：[luci-app-rgbled_1.7-1_all.ipk](packages/luci-app-rgbled_1.7-1_all.ipk)

适配 AW9523B，单独控制 4 颗 RGB 灯珠。支持六位 RGB 十六进制颜色、三原色滑杆、0–100% 亮度、总开关以及一次应用全部灯珠配置。每颗灯可绑定 2.4 GHz、5 GHz、2.4+5 GHz、LAN1、LAN2 或 5G Modem 流量；有连续数据包时以约 1 ms 亮/灭交替，空闲时常亮。也可绑定单个温度传感器或 Max/Avg/Min 温度规则：低于 80°C 熄灭、80–90°C 每 500 ms 闪烁、90–100°C 常亮、超过 100°C 每 10 ms 闪烁。

### Modem Control

文件：[luci-app-modem-control_1.3-2_all.ipk](packages/luci-app-modem-control_1.3-2_all.ipk)

面向 Quectel `ql-CM` 数据连接。页面显示网络类型、Band、运营商、Modem 型号、RSRP/SINR/RSRQ 及 Very good–Very poor 信号等级、PCI、EARFCN/NR-ARFCN、小区 ID、IMEI、IMSI、ICCID、QCI/5QI 优先级、IPv4 和格式化后的 IPv6。支持持久化 APN、LTE/NSA/SA 频段复选锁定与解除、短信收件箱（AT 回退方案）以及 AT 指令终端。

### Temperature Sensors

文件：[luci-app-temperature-monitor_1.0-4_all.ipk](packages/luci-app-temperature-monitor_1.0-4_all.ipk)

自动枚举 `/sys/class/thermal/thermal_zone*` 中的全部温度传感器，每秒刷新传感器名称、区域和当前温度，并显示传感器数量及平均温度。页面样式与本机旧版 LuCI 白色后台保持一致。

## 局域网测速网页

源码位于 [`speedtest/`](speedtest/)，直接测试终端与路由器之间的局域网链路，不依赖互联网测速节点。页面提供延迟、下载和上传速度测试，独立 HTTP 服务默认监听 `192.168.1.1:8090`，避免被 LuCI 登录重定向拦截。

## 安装插件

把 IPK 上传到路由器后执行：

```sh
opkg install /tmp/luci-app-rgbled_1.7-1_all.ipk
opkg install /tmp/luci-app-modem-control_1.3-2_all.ipk
opkg install /tmp/luci-app-temperature-monitor_1.0-4_all.ipk
rm -f /tmp/luci-indexcache
/etc/init.d/uhttpd restart
```

这些包针对本项目所用的旧版 LuCI/Chaos Calmer 环境制作；其他 OpenWrt 版本可能需要调整 Lua API、页面模板或硬件接口。

## 固件组件

- 下载：[Horizon_HZ51_OpenWrt_firmware-components.zip](firmware/Horizon_HZ51_OpenWrt_firmware-components.zip)
- 解压后的文件位于 [`firmware/components/`](firmware/components/)：`kernel.bin`、`ubi_rootfs.bin`、`wifi_fw.bin`、`bt_fw.bin`。
- 所有发布文件的校验值见 [`SHA256SUMS`](SHA256SUMS)。

公开包刻意排除了 `rootfs_data`、ART、设备配置、工厂分区和完整闪存镜像，避免泄露 Wi-Fi 密码、SSH 私钥、MAC、IMEI/IMSI/ICCID、APN、短信或校准数据。因此它是供开发和恢复分析使用的固件组件包，不能直接上传到 LuCI，也不应盲目写入其他设备。

## 免责声明

使用这些固件组件和插件存在变砖、失去网络连接或硬件行为异常的风险。操作前请保留串口救援手段和本机完整备份。移动网络频段、发射功率及运营商功能须遵守所在地法规和运营商规则。
