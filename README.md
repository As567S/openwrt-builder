# 我的 OpenWrt 自动编译（x86_64）

基于 GitHub Actions：OpenWrt 官方一发布新的 stable 版本，就自动编译一份属于自己的固件，并发布到本仓库的 Releases。

## 工作方式

- **自动**：每天 03:17 (UTC) 检查一次官方最新 stable tag（如 v25.12.0）。若本仓库还没发布过这个版本，就自动开编；编过就跳过。
- **手动**：仓库页面 Actions →「Build OpenWrt (x86_64)」→ Run workflow。可留空（= 最新 stable），也可填指定版本 tag；勾选 force 可重编已发布过的版本。
- **产物**：在 Releases 里，tag 就是 OpenWrt 版本号。含固件镜像（combined / combined-efi 的 .img.gz）、校验文件、本次构建实际用的 `firmware.config`，以及全部 ipk 包合集 `packages-x86_64.tar.gz`。

## 仓库结构

| 文件 | 作用 |
|---|---|
| `.config` | 固件配置（目标机 x86_64 generic + 预装包清单），换包就改它 |
| `custom-packages.txt` | 自定义包清单，一行一个 Git 仓库，构建时克隆进 `package/` 一起编 |
| `extra-config.txt` | 强制启用的配置行（自定义包必须在这里打开，如 `CONFIG_PACKAGE_luci-app-geph5=y`） |
| `.github/workflows/build-openwrt.yml` | 构建流水线，一般不用动 |

## 已包含的自定义包

- **luci-app-geph5**（迷雾通 5 的 LuCI 界面）：注意它的 geph5-client 是编译时从 crates.io 拉 Rust 源码现场交叉编译的，所以流水线里装了 Rust 工具链（x86_64-unknown-linux-musl target），首次编译会比普通包慢一些。

## 加新包的方法

1. 把包的 Git 仓库地址加进 `custom-packages.txt`；
2. 把 `CONFIG_PACKAGE_<包名>=y` 加进 `extra-config.txt`；
3. 提交推送，下次构建生效（可手动 Run workflow 立即验证）。

## 注意

- 构建耗时约 2–4 小时（GitHub 免费 runner），公开仓库 Actions 额度充足。
- `.config` 是基于较新版本（内核 6.12）生成的；构建新版本时流水线会先 `make defconfig` 自动对齐符号，个别包在新版本中改名/下线时可能被丢弃，对照 Release 里的 `firmware.config` 即可确认最终生效的清单。
