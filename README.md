# WRT 项目分析文档

## 一、项目概述

本项目是一个基于 OpenWRT/ImmortalWRT 的自定义固件编译系统，用于编译适配多种路由器的定制固件。项目地址: https://github.com/ZqinKing/wrt_release

## 二、源码基础

### 2.1 基础源码

项目基于 **ImmortalWRT** 进行定制开发。ImmortalWRT 是 OpenWRT 的一个 fork，专注于提供更稳定的固件和更新的软件包。

主要配置参数（来自 `wrt_core/compilecfg/*.ini`）:
- **REPO_URL**: https://github.com/immortalwrt/immortalwrt.git
- **BUILD_DIR**: 编译输出目录（如 `immwrt`, `libwrt`, `imm-nss` 等）
- 可指定特定 COMMIT_HASH 进行精确定位

### 2.2 Feed 源

项目使用以下第三方 Feed 源:

| Feed 名称 | 仓库地址 | 用途 |
|-----------|----------|------|
| small8 (jell) | https://github.com/kenzok8/jell | 第三方插件集合 |
| openwrt-passwall | https://github.com/Openwrt-Passwall/openwrt-passwall;main | PassWall 代理 |
| openwrt_bandix | https://github.com/timsaya/openwrt-bandix.git;main | Bandix 代理 |
| luci_app_bandix | https://github.com/timsaya/luci-app-bandix.git;main | Bandix LuCI |

## 三、自定义修改

### 3.1 新增软件包

项目添加了以下自定义软件包（来自 packages.sh）:

| 软件包 | 仓库地址 | 功能说明 |
|--------|----------|----------|
| luci-app-athena-led | https://github.com/NONGFAH/luci-app-athena-led.git | 京东云雅典娜 LED 控制 |
| luci-app-timecontrol | https://github.com/sirpdboy/luci-app-timecontrol.git | 时间控制 |
| luci-app-quickfile | https://github.com/sbwml/luci-app-quickfile.git | QuickFile 文件管理 (nginx) |
| smartdns | https://github.com/ZqinKing/openwrt-smartdns.git | 智能 DNS |
| luci-app-adguardhome | https://github.com/ZqinKing/luci-app-adguardhome.git | AdGuard Home |
| luci-lib-docker | https://github.com/lisaac/luci-lib-docker.git | Docker LuCI 库 |
| luci-theme-argon | https://github.com/ZqinKing/luci-theme-argon.git | Argon 主题 |

### 3.2 第三方插件集成

三方插件整合自: https://github.com/kenzok8/small-package.git

**代理/VPN 类**:
- OpenClash - Clash 代理客户端
- HomeProxy - 代理客户端
- PassWall - 代理客户端
- V2Ray
- Sing-Box
- Trojan Plus
- naiveproxy
- Hysteria
- TUIC

**DNS/广告过滤类**:
- MosDNS - DNS 解析
- SmartDNS - 智能 DNS
- AdGuardHome - 广告拦截

**应用类**:
- LuCI App Store (iStore)
- Docker Manager (Dockerman)
- DiskMan - 磁盘管理
- Open App Filter (OAF) - 应用过滤
- Policy Based Routing (PBR) - 策略路由
- Cloudflare Speed Test
- NetData - 系统监控

### 3.3 补丁文件

项目在 `wrt_core/patches/` 目录下包含以下补丁:

| 补丁文件 | 说明 |
|----------|------|
| 100-smartdns-optimize.patch | SmartDNS 优化 |
| 001-fix-provides-version-parsing.patch | Opkg 版本解析修复 |
| 999-chanage-default-leaseduration.patch | Miniupnpd 租约时间修改 |
| 999-libubox-demote-format-nonliteral.patch | libubox 修复 |

**OpenSSL 补丁** (`wrt_core/patches/openssl/patches/`):
- 100-Configure-afalg-support.patch - AFAlg 支持
- 110-openwrt_targets.patch - OpenWRT 目标配置
- 120-strip-cflags-from-binary.patch - 移除 CFLAGS
- 130-dont-build-fuzz-docs.patch - 不构建 fuzz 文档
- 140-allow-prefer-chacha20.patch - 允许优先 Chacha20
- 150-openssl.cnf-add-engines-conf.patch - 添加 engines 配置
- 500-e_devcrypto-default-to-not-use-digests-in-engine.patch
- 510-e_devcrypto-ignore-error-when-closing-session.patch

### 3.4 系统级修改

项目对系统进行了以下修改（来自 system.sh）:

| 修改项 | 说明 |
|--------|------|
| 默认主题 | 设为 argon 主题 |
| 默认 LAN IP | 可自定义修改 |
| CPU 使用率脚本 | 适配 qualcommax 和 mediatek 平台 |
| DNS 配置 | 修改 dnsmasq 配置 |
| NSS 诊断脚本 | 自定义 nss_diag.sh |
| WiFi 配置 | 预设 WiFi 默认配置 |
| 启动任务 | 自定义 custom_task 启动项 |
| 备份配置 | 添加 lucky 到 sysupgrade.conf |
| PBR 配置 | 添加 CMCC 特定配置 (pbr.user.cmcc, pbr.user.cmcc6) |

### 3.5 软件包移除

项目在编译前会移除以下软件包（来自 packages.sh）:

**LuCI 应用**:
- luci-app-passwall (使用三方版本)
- luci-app-ddns-go
- luci-app-rclone
- luci-app-ssr-plus
- luci-app-vssr
- luci-app-daed
- luci-app-dae
- luci-app-alist
- luci-app-homeproxy
- luci-app-haproxy-tcp
- luci-app-openclash
- luci-app-mihomo
- luci-app-appfilter
- luci-app-msd_lite
- luci-app-unblockneteasemusic

**网络包**:
- haproxy, xray-core, xray-plugin, dns2socks
- alist, hysteria, mosdns, adguardhome
- ddns-go, naiveproxy, shadowsocks-rust
- sing-box, v2ray-core, v2ray-geodata
- v2ray-plugin, tuic-client, chinadns-ng
- ipt2socks, tcping, trojan-plus
- simple-obfs, shadowsocksr-libev
- dae, daed, mihomo, geoview
- open-app-filter, msd_lite

## 四、支持的设备

项目支持以下设备编译:

### 4.1 京东云 (JD Cloud)
| 设备 | 编译目标 | 平台 |
|------|----------|------|
| 雅典娜(02) | jdcloud_ipq60xx_immwrt | ipq60xx |
### 4.2 其他
| 设备 | 编译目标 | 平台 |
|------|----------|------|
| X64 | x64_immwrt | x86_64 |

## 五、配置文件说明

### 5.1 核心配置文件

| 文件 | 说明 |
|------|------|
| wrt_core/deconfig/*.config | 设备默认配置文件 |
| wrt_core/compilecfg/*.ini | 编译参数配置文件 |
| wrt_core/deconfig/nss.config | NSS 驱动配置 (IPQ60xx/IPQ807x) |
| wrt_core/deconfig/proxy.config | 代理相关配置 |
| wrt_core/deconfig/compile_base.config | 基础编译配置 |

### 5.2 编译配置 (compile_base.config)

默认启用的功能:
- ccache 编译加速
- telnet 支持
- WireGuard 协议
- 中继 (Relay) 支持
- libopenssl-legacy
- dnsmasq-full
- Coremark 性能测试
- iptables-nft / ip6tables-nft

默认启用的 LuCI 应用:
- luci-app-istorex
- luci-app-lucky
- luci-app-mosdns
- luci-app-oaf
- luci-app-pbr
- luci-app-smartdns
- luci-app-ttyd
- luci-app-upnp
- luci-app-wol

## 六、编译流程

### 6.1 主流程 (build.sh)

```
1. 读取设备配置 (.config) 和编译参数 (.ini)
2. 调用 wrt_core/update.sh 克隆/更新源码
3. 应用配置文件:
   - 设备配置
   - NSS 配置 (IPQ60xx/IPQ807x)
   - 基础编译配置
   - 代理配置
   - Docker 依赖配置
4. 执行 make defconfig
5. 下载依赖: make download -j$(nproc*2)
6. 编译固件: make -j$(nproc+1)
7. 整理固件到 firmware/ 目录
```

### 6.2 模块流程

| 模块 | 文件 | 功能 |
|------|------|------|
| general.sh | 通用模块 | 克隆仓库、清理、feeds 重置 |
| feeds.sh | Feeds 模块 | 更新/安装 feeds 源 |
| packages.sh | 软件包模块 | 安装、更新、移除软件包 |
| system.sh | 系统模块 | 应用补丁、修改配置 |

## 七、项目结构

```
wrt_release/
├── build.sh                    # 主编译脚本
├── WRT-README.md              # 本文档
├── wrt_core/
│   ├── update.sh              # 源码更新脚本
│   ├── pre_clone_action.sh    # 预克隆操作
│   ├── modules/               # 核心模块
│   │   ├── general.sh         # 通用模块
│   │   ├── feeds.sh           # Feeds 模块
│   │   ├── packages.sh        # 软件包模块
│   │   ├── system.sh          # 系统模块
│   │   └── cups.sh            # CUPS 模块
│   ├── patches/               # 补丁目录
│   │   ├── openssl/           # OpenSSL 补丁
│   │   ├── lucky_*.tar.gz     # lucky 二进制
│   │   ├── cpuusage           # CPU 使用率脚本
│   │   ├── smp_affinity       # CPU 亲和性脚本
│   │   └── *.patch            # 各类补丁
│   ├── deconfig/              # 设备配置文件
│   │   ├── *.config           # 各设备配置
│   │   ├── nss.config         # NSS 配置
│   │   ├── proxy.config       # 代理配置
│   │   └── compile_base.config # 基础配置
│   └── compilecfg/            # 编译参数配置
│       └── *.ini              # 各设备编译参数
└── firmware/                   # 编译输出目录
```

## 八、特性亮点

1. **多平台支持**: 支持 Qualcomm IPQ60xx/IPQ807x、MediaTek、x86_64 等多种平台
2. **代理整合**: 集成 Clash、V2Ray、Sing-Box、PassWall 等多种代理客户端
3. **广告拦截**: 内置 AdGuardHome、MosDNS、SmartDNS
4. **Docker 支持**: 完整的 Docker 和 Dockerman 管理界面
5. **主题定制**: 使用 Argon 主题，界面美观
6. **性能优化**: 
   - NSS 驱动优化
   - CPU 亲和性配置
   - ccache 加速编译
7. **自动更新**: 支持在线更新和 opkg 扩展

## 九、注意事项

1. **首次编译**需要执行环境准备脚本安装依赖
2. **OAF 功能**需要在 LuCI 中手动启用
3. **PassWall** 会自动禁用 uhttpd 依赖（使用 nginx 替代）
4. **Docker 依赖**会自动添加到配置中
5. 编译过程中会自动下载必要的 Geodata 文件