# 安装 GODOFTUN V1.0.0

[English](INSTALLATION.md) · [فارسی](INSTALLATION.fa.md) · [Русский](INSTALLATION.ru.md) · [首页](../README.zh-CN.md)

本指南介绍如何在两台服务器上安装一对隧道端点。请先配置境外服务器，再配置伊朗服务器。在替换正在运行的服务之前，先使用独立服务器进行测试。

## 连接路径

```text
Client application
  -> Iran public IP and TCP entry port
  -> GODOFTUN origin
  -> foreign domain through Cloudflare WSS
  -> foreign Nginx and GODOFTUN egress
  -> dedicated application inbound on 127.0.0.1
```

支持的配置固定为：境外域名开启 Cloudflare Proxy ON、使用 WSS、伊朗端通过 IP 直接接入。没有伊朗域名、Arvan、境外直连、HTTP/2 或 XHTTP 选项。这是 TCP 隧道，不是 UDP 隧道，也不是覆盖整台设备的 VPN。Xray 或面板管理的应用仍然需要自己的协议、凭据和客户端配置。GODOFTUN 不是 Cloudflare Tunnel 或 `cloudflared` 产品。

## 开始之前

- 准备两台使用 systemd 的 Ubuntu 或 Debian 服务器，通过 `sudo` 获得 root 权限，并具备公网 IPv4 地址。安装器支持 amd64/x86_64 和 arm64/aarch64，不支持 32 位系统。macOS 不是安装目标。
- 准备一个由你控制的独立境外子域名，例如 `tunnel.example.com`，并按照 [Cloudflare 指南](CLOUDFLARE.zh-CN.md)配置。此名称仅为示例，不是可用的服务端点；请替换为自己的域名。
- 在境外服务器上准备一个专用应用入站，监听 `127.0.0.1` 上未被占用的 TCP 端口。例如，只有实际在该端口创建了入站时，才可使用 `18082`。不要修改无关的面板入站。
- 准备源站证书和私钥，或者确保安装器能够完成 Let's Encrypt HTTP 验证。只有边缘证书是不够的。
- 确保能够访问用于安装缺失工具的软件包仓库、用于验证代理状态的 Cloudflare IP 地址范围 API，以及境外域名所选的 HTTPS 端点。ECH 还需要可用的 HTTPS DNS 发现。离线复制安装器并不能消除这些网络要求。
- 保持服务器时钟同步，并确保现有 SSH 访问不会受到防火墙变更的影响。

安装器会检查操作系统系列，但不代表已经验证每个 Ubuntu/Debian 版本。请参阅[已测试范围](../VALIDATION.md)。使用仍受支持并保持更新的系统软件包。两台服务器都不需要 Go 编译器。

## 验证下载文件

解压完整的公开 ZIP 包，进入其中的 `sik-dpi` 文件夹，并将 `SHA256SUMS` 与其他文件保存在一起。在 Linux 上运行：

```bash
sha256sum -c SHA256SUMS
bash -n GODOFTUN-V1.0.0.sh
bash GODOFTUN-V1.0.0.sh preview
```

在 macOS 上，将第一条命令替换为 `shasum -a 256 -c SHA256SUMS`；菜单预览也可以在不安装的情况下运行。要完成全部校验，清单中列出的每个文件都必须存在。校验和可以发现文件相对于清单的变化，但无法证明被恶意替换的清单是真实的。请从项目所有者可信的发布渠道同时获取文件和清单。

`preview` 命令仅显示菜单。不带参数运行时，会打开交互式主菜单。不要直接通过高权限 shell 执行未经验证的下载内容。

## 网络端口

| 位置 | 所需访问权限 |
|---|---|
| 境外公网接口 | 所选的 HTTPS TCP 端口，通常为 443；安装的 HTTP 站点和自动证书验证还需要 TCP 80 |
| 境外回环接口 | 专用应用入站和安装器选择的私有 WSS 后端；不要将这些端口对公网开放 |
| 伊朗公网接口 | 你选择的客户端 TCP 接入端口，例如未被占用的 21000 和 21001 |
| 伊朗出站连接 | DNS、前置检查所需的 HTTPS，以及境外域名所选的 HTTPS 端口 |
| 两台服务器 | 现有 SSH 访问，以及必要的软件包和时间同步服务 |

公网 HTTPS 端口与伊朗客户端端口是不同的设置，无需一致。同一隧道中的多个伊朗端口都会到达同一个已配置的境外应用目标。如果需要另一个目标，请创建另一条具备独立资源的命名隧道。如果 UFW 防火墙已经启用，安装器会添加所需的 TCP 规则；它不会启用 UFW，也不会配置服务商的防火墙。请检查这两处规则。不要清空共享防火墙的规则。

## 配置境外服务器

先创建应用入站，并完成 [DNS 和证书准备](CLOUDFLARE.zh-CN.md)。然后运行：

```bash
sudo bash GODOFTUN-V1.0.0.sh foreign
```

按照提示填写：

1. 选择隧道 ID 和便于识别的名称。ID 长度为 1–48 个字符，可使用小写字母、数字和单个连字符，必须以字母或数字开头，且不能包含连续的连字符。新隧道应使用新的 ID。
2. 选择开启或关闭 MUX。除非有明确原因，否则先使用默认值；请参阅[配置指南](CONFIGURATION.zh-CN.md)。
3. 输入已经开启橙色云朵代理的境外域名，以及境外服务器真实的 IPv4 地址。公网 DNS 应返回 Cloudflare 地址，而不是该源站地址。
4. 按优先级顺序输入 HTTPS 端口。建议先使用 `443`。仅接受列出的、与 Cloudflare 兼容的 HTTPS 端口。
5. 输入专用入站的实际端口。安装器会检查 `127.0.0.1:PORT`，并另外选择一个空闲的私有 WSS 后端。
6. 选择浏览器 TLS 配置和填充强度，默认值为 `0.22`。
7. 选择自动申请 Let's Encrypt 证书，或使用现有证书/私钥及本地信任证书包。确认安装之前，先解决预检错误。

启动后，安装器会检查本地 HTTPS、经过 HMAC 身份验证的 WebSocket 升级（不是完整的 V1 质询-响应过程）和 CDN 路径。请妥善保密生成的配对码：其中包含连接信息和共享令牌。Base64 不是加密。不要将配对码放在公开 issue、截图或 GitHub 文件中。

## 配置伊朗服务器

将同一份已经校验的安装器传到伊朗服务器；[运维指南](OPERATIONS.zh-CN.md)介绍了无法访问 GitHub 时的传输方法。运行：

```bash
sudo bash GODOFTUN-V1.0.0.sh iran
```

1. 回答是否已经注册了其他隧道。这个回答并不意味着授权替换已有隧道。
2. 粘贴境外配对码。留空会进入手动填写模式；首次安装建议使用配对码，以避免配置不一致。
3. 按提示选择本地隧道 ID/名称，然后输入伊朗服务器的公网 IPv4 地址。系统不会要求伊朗域名。
4. 输入一个或多个未被占用的客户端端口，用逗号分隔，例如 `21000,21001`。
5. 选择会话上限和填充设置。MUX 和传输方式会从配对端点导入。
6. 选择 ECH OFF、严格的 ECH ON，或 ECH OFF 加实验性的 ClientHello Fragment。ECH 与 Fragment 是互斥选项，不是组合模式。请参阅[配置指南](CONFIGURATION.zh-CN.md)。
7. 以十进制 GB 设置整个使用周期的总配额。`0` 表示无限制，上传和下载合并计量。
8. 预检通过后再确认。安装器会验证传输会话，但不会验证应用的凭据或实际载荷。

如果输入或验证失败，请阅读错误信息，并重试提示的步骤。不要通过关闭证书验证或改用不受支持的传输方式来绕过检查。

## 连接并测试真实客户端

在每台服务器上查找其隧道 ID：

```bash
sudo godoftunctl list
sudo godoftunctl status TUNNEL_ID
sudo godoftunctl doctor TUNNEL_ID
```

将 `TUNNEL_ID` 替换为列表中显示的本地 ID。在伊朗服务器上，`sudo godoftunctl info TUNNEL_ID` 会显示客户端指引。请勿公开这些输出。将客户端的连接地址设置为伊朗 IP 和分配给它的接入端口，同时保留境外专用入站的协议、UUID/密码和应用层安全设置。如果该应用使用 TLS，其 SNI/证书要求仍然适用。不要将应用的 TLS 与隧道外层 WSS 的 TLS 混淆。

为用户提供服务之前，请在实际路径上确认以下所有事项：

- 经过真实身份验证的客户端能够通过每个计划使用的伊朗端口上传和下载。
- 并发连接和较大数据传输正常，不会发生响应截断。
- 计划内重启服务后，新连接能够恢复；不保证现有数据流无缝存活。
- 选择严格 ECH 时，握手确实接受 ECH；单独测试 Fragment，不要假定它一定有效。
- 有限额的测试客户达到配额后会被正确阻止；提高配额不会抹除已记录的用量。
- 诊断、证书续期以及服务器重启后的访问都能正常工作。进程显示为 `active` 或能够打开普通网页，都不足以证明整个应用正常。

请将测试结果保密，在求助前移除凭据。接下来请阅读[运维](OPERATIONS.zh-CN.md)、[故障排查](TROUBLESHOOTING.zh-CN.md)和[安全](SECURITY.zh-CN.md)指南。
