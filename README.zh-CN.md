# GODOFTUN V1.0.0

多端口 TCP 隧道：境外域名启用 Cloudflare 代理，用户通过伊朗服务器的 IP 和端口连接。

[English](README.md) · [فارسی](README.fa.md) · [Русский](README.ru.md) · [社区 SIKDPI](https://t.me/SIKDPI)

## 文档

运行安装程序前，请先阅读安装和 Cloudflare 配置指南。示例中的占位值必须替换为自己的服务器信息。

| 指南 | 内容 |
|---|---|
| [安装](docs/INSTALLATION.zh-CN.md) | 前提条件、校验文件、境外和伊朗服务器安装、客户端配置与验收清单 |
| [Cloudflare](docs/CLOUDFLARE.zh-CN.md) | DNS、开启代理、端口、WebSocket、源站证书、续期与 ECH |
| [配置](docs/CONFIGURATION.zh-CN.md) | 默认值、限制、MUX、填充、会话、流、内存、TLS 与两端同步修改 |
| [运维](docs/OPERATIONS.zh-CN.md) | 命令参考、客户端口、配额、监控、文件传输、备份、升级与删除 |
| [故障排查](docs/TROUBLESHOOTING.zh-CN.md) | Doctor、常见故障、恢复与安全的支持报告 |
| [安全](docs/SECURITY.zh-CN.md) | 可见代码、机密信息、信任边界与发布限制 |

全部六篇指南均提供以上四种语言。命令和程序的终端界面仍为英文。[验证结果](VALIDATION.md) 区分已完成的本地测试与尚未执行的 Linux 和真实 Cloudflare 链路测试。

## 下载与运行

解压完整的公开 ZIP 并进入 `sik-dpi` 文件夹。使用同一可信发行包中的独立安装程序 `GODOFTUN-V1.0.0.sh`。在 Linux 上先校验全部文件：

```bash
sha256sum -c SHA256SUMS
```

在 macOS 上使用 `shasum -a 256 -c SHA256SUMS`。如果文件及校验清单都由攻击者控制，校验和并不能证明来源可信。

只预览菜单，不安装：

```bash
bash GODOFTUN-V1.0.0.sh preview
```

在干净的测试服务器上，先安装境外端：

```bash
sudo bash GODOFTUN-V1.0.0.sh foreign
```

随后在伊朗服务器运行，并粘贴机密配对码：

```bash
sudo bash GODOFTUN-V1.0.0.sh iran
```

安装程序内含 Linux amd64 和 arm64 可执行文件，目标服务器无需 Go。目标环境为使用 systemd、具有公网 IPv4 的 Ubuntu/Debian。Nginx 等依赖可能仍需下载。复制安装程序不等于完全离线部署：DNS、Cloudflare 代理验证及隧道链路必须可达。本程序仅传输 TCP，不是 UDP 隧道或整机 VPN。

## 功能

- 境外域名必须开启 Cloudflare 代理；仅使用 WSS。
- 多个客户端端口和命名隧道。
- MUX 开关、受限会话数、流数和保留缓冲区内存限制。
- 字节填充默认值为 0.22；支持严格 ECH 或 ClientHello 分片。
- 两端同步配置、客户端口配额与诊断。
- 青色终端菜单，分为 Tunnels、Tuning、Customers 和 Doctor，安装按步骤进行。

不提供 Arvan、境外直连模式或伊朗端域名。V1 保留固定的 512 KiB 流量控制窗口，不含 3.5 的自适应窗口控制器。请勿混用 V1 与 3.x 的安装或配对信息。

## 终端界面

不带参数运行安装程序可打开主菜单；安装后使用 `manage`。`preview` 无需 root，不检查网络、不修改服务器，也不显示虚构的实时状态。

通过 `GODOFTUN_THEME=dark`（默认）或 `GODOFTUN_THEME=light` 选择适合深色或浅色终端的强调色。`NO_COLOR=1` 禁用颜色；文件输出和 `TERM=dumb` 自动使用纯文本。菜单会根据宽度换行；`COLUMNS=60` 设置为 60 列。原有操作编号与确认步骤保持不变。

## 源码与安全

这是**二进制分发包**，不是开源仓库。包中不含 Go 实现源码；安装程序及嵌入的 Bash/Python 管理代码可读，编译后的程序也能被分析。项目不声称能完全隐藏源码或阻止逆向工程。

向用户部署前，请阅读[验证结果](VALIDATION.md)，在自己的干净服务器上完成真实 Linux 安装和 Cloudflare 链路测试。不要公开 token、私钥、配对或编辑码以及客户用量文件。

项目自身的分发条款仍需所有者批准。依赖许可证位于 `THIRD_PARTY_LICENSES/`。参见[致谢](ACKNOWLEDGMENTS.md)。
