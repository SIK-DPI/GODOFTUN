# GODOFTUN V1.0.0 故障排查

[English](TROUBLESHOOTING.md) · [فارسی](TROUBLESHOOTING.fa.md) · [简体中文](TROUBLESHOOTING.zh-CN.md) · [Русский](TROUBLESHOOTING.ru.md)

[项目说明](../README.zh-CN.md) · [安装](INSTALLATION.zh-CN.md) · [配置](CONFIGURATION.zh-CN.md) · [运维](OPERATIONS.zh-CN.md) · [Cloudflare](CLOUDFLARE.zh-CN.md) · [安全](SECURITY.zh-CN.md)

从受影响隧道的第一个失败阶段开始排查。重装、更换证书或重启所有服务可能掩盖最初的原因。这些检查不保证特定速度、可用性或绕过网络过滤的能力。

将下列命令中的 `TUNNEL_ID` 替换为 `list` 显示的隧道 ID。在 GODOFTUN 日志中，伊朗端角色为 `origin`，境外服务器为 `egress`。Cloudflare 所说的“origin server”指境外 HTTPS 服务器，而不是 GODOFTUN 的伊朗端角色。

## 收集基本信息

```bash
sudo godoftunctl list
sudo godoftunctl status TUNNEL_ID
sudo godoftunctl health TUNNEL_ID
sudo godoftunctl ports TUNNEL_ID
sudo godoftunctl doctor TUNNEL_ID --offline
sudo godoftunctl doctor TUNNEL_ID --timeout 8
sudo journalctl -u godoftun@TUNNEL_ID --since "15 minutes ago" -n 120 --no-pager
```

在两台服务器上分别运行 Doctor。`ports` 显示已登记的端口，并不能证明进程正在监听这些端口。`status`、`health` 和 `test` 可用于检查状态，但服务处于活动状态或隧道连接通过认证，不等于 Xray 客户端已成功上传和下载。分享日志前请先检查内容。

## Doctor 命令与结果范围

已安装的管理命令接受以下选项，放在隧道 ID 后面。`scan` 是 `doctor` 的别名。

| 选项 | 含义 |
|---|---|
| `--offline` | 解析所选配置，不发起网络请求。 |
| `--endpoints HOST:PORT,...` | 测试明确指定的候选端点。支持 IP 或域名，包括 `[IPv6]:PORT`；不接受 URL、CIDR、地址范围或端口范围。省略端口时使用配置中的第一个境外端口。 |
| `--timeout N` | 每个阶段的秒数，整数 1–20，默认 8。核心认证尝试也有有限超时。这不是整份多端点报告的总时间限制。 |
| `--output FILE` | 创建权限为 `0600` 的脱敏 JSON 报告。父目录必须已存在；不会覆盖已有文件。 |
| `--binary PATH` | 使用匹配版本的 GODOFTUN 可执行文件，默认 `/usr/local/bin/godoftun`。不要传入 `run.sh`。 |
| `--help` | 显示诊断工具的实际命令帮助。 |

例如，比较已配置域名的两个端口：

```bash
sudo godoftunctl doctor TUNNEL_ID --endpoints tunnel.example.com:443,tunnel.example.com:8443 --timeout 8
sudo godoftunctl doctor TUNNEL_ID --output /root/godoftun-TUNNEL_ID-report.json
```

只填写你打算测试的端点。测试候选端点时，Doctor 保留已配置的 Host、SNI、路径和认证信息，不改写隧道配置。明确指定 IP 进行测试只是诊断操作，不代表支持直接通过 IP 部署。报告最多测试 16 个候选端点，DNS 解析得到的地址也计入上限。域名返回多个地址时，请检查 `candidate_limit_reached`。

在伊朗端，默认测试配置中的境外域名/端口条目。在境外服务器上，默认测试 `127.0.0.1` 的本机 HTTPS 和 Cloudflare 边缘端点，二者都只使用配置中的**第一个**境外端口。其他端口需要明确指定。

| 结果 | 可以确认什么 |
|---|---|
| `configuration-checked-not-network-tested` | 离线配置解析通过，未测试网络。退出码 0。 |
| `carrier-authenticated` | 至少一次真实的 WSS/V1 认证通过，且报告中没有失败阶段。退出码 0。 |
| `partial` | 某个连接认证通过，但另一个阶段或端点失败。退出码 1。 |
| `failed` | 没有候选端点通过认证。退出码 1。 |
| `configuration-error` | 配置未被接受，或无法读取本地输入文件、安全创建报告文件。退出码 2。 |

`actual-carrier` 使用真实核心执行 TLS 验证、浏览器配置、ECH/Fragment 握手、WebSocket 升级和 V1 质询响应认证，不会打开到 Xray 目标的流。因此，报告中的 `target_payload_tested` 和 `cdn_origin_policy_verified` 明确为 false。推荐端点仅代表单次握手时间最短，不是吞吐量或可靠性测试结果。

## 安装在创建隧道前停止

安装程序要求 root 权限、Ubuntu 或 Debian、systemd，以及受支持的 Linux amd64 或 arm64 架构。服务器无需安装 Go 即可运行包内可执行文件。缺失的工具或依赖仍可能需要可访问的软件包仓库。请读取具体缺失命令或软件包错误；安装程序不会自动修复已安装但损坏的 Nginx。

出现 `flock is required` 时，请通过正常的服务器维护流程提供发行版的 `util-linux` 依赖。出现 `Another GODOFTUN installation is running` 时，等待现有操作完成；不要删除锁或启动第二次安装。

端口冲突时，先确认占用者，再进行修改：

```bash
sudo ss -ltnp 'sport = :443'
sudo nginx -t
```

将 `443` 替换为错误报告中的端口。通过安装程序或编辑器选择空闲端口，或协调现有服务的配置。`nginx -t` 失败可能涉及其他站点；不要删除无关站点来让测试通过。如果域名已由另一个 Nginx 站点使用，请为隧道选择独立主机名。Go WebSocket 后端应保持仅监听回环地址。不要为了满足新隧道的目标检查而修改现有 Native Reverse 入站；请按照[安装指南](INSTALLATION.zh-CN.md)创建独立入站。

## DNS 未解析到 Cloudflare

此安装程序要求境外主机名**只**解析到经过验证的 Cloudflare 代理地址。DNS-only 记录、A/AAAA 答案中包含源站 IP、过期的解析缓存，或无法下载 Cloudflare 官方 IP 范围，均可能使预检查停止。运行时连接域名、SNI 和 WebSocket Host 必须一致。

```bash
getent ahosts tunnel.example.com
timedatectl status
```

检查 A 和 AAAA 记录、所选主机名、Proxy ON 状态，以及 Cloudflare 中配置的源站地址。等待 DNS 变更传播后，重新运行失败的检查。无法获取官方 IP 范围表示验证未完成，并不单独证明 DNS 记录错误。不要把伊朗端连接配置中的域名替换为境外 IP，也不要禁用代理验证。参见 [Cloudflare](CLOUDFLARE.zh-CN.md)。

安装程序接受的境外 HTTPS 端口为 `443`、`2053`、`2083`、`2087`、`2096` 和 `8443`。伊朗客户端端口、境外 HTTPS 端口、Go 回环后端端口和 Xray 目标端口各有不同用途。

## 定位失败的网络阶段

| 现象或消息 | 下一步检查 |
|---|---|
| `DNS lookup failed` | DNS 记录与解析器可达性。此次解析失败后没有进行 TLS 测试。 |
| `[tcp-connect]`、connection refused | 所选监听服务、地址/端口及明确的网络拒绝。更换证书无法打开 TCP 监听端口。 |
| Timeout 或 `deadline` | 报告中的阶段、路由、服务器负载、监听状态，以及相关主机/提供商防火墙规则。单次超时无法确定是过滤、令牌错误还是证书问题。 |
| `[tls-config]`、`[tls-handshake]`、`x509`、fingerprint mismatch | 信任来源、证书主机名/有效期、正确的 TLS 链路，以及所配置的证书固定指纹。 |
| `[tls-alpn] expected http/1.1` | 所选端点及 TLS 前端。此连接需要 HTTP/1.1 WebSocket 升级。 |
| WebSocket `404` | 精确的私有路径、Host、转发的 Authorization/Upgrade 头、共享令牌与系统时间。预认证被拒绝时也会故意返回 404。 |
| `403` 或 `429` | 特定路由的 CDN 挑战、WAF、访问控制或速率限制。这不能证明令牌错误。 |
| `502`、`503`、`504`、`525` 或 `526` | Cloudflare 到境外服务器的地址/端口、源站 HTTPS/证书、Nginx 及回环后端状态。状态码本身不能确定具体原因。 |
| `expected challenge`、`egress rejected token`、unsupported protocol version | 两端 V1 版本及配对认证配置是否匹配。WSS 升级成功还不够。 |

隧道无法完成交互式浏览器挑战。请结合事件证据检查 Cloudflare 中精确主机名/路径的策略，不要关闭整个账户的安全设置或服务器防火墙。公共伪装页面 `/` 能响应，并不证明私有隧道路由正常。

## 一段 TLS 链路成功，另一段失败

伊朗到 Cloudflare 边缘，以及 Cloudflare 到境外 HTTPS 服务器，是独立的 TLS 连接。Doctor 还会检查境外服务器的本机 HTTPS。边缘证书无法修复缺失、不匹配或过期的源站证书。使用 Cloudflare Origin CA 证书时，本机源站检查需要相应的 CA 证书包；它不是普通客户端默认信任的公开证书。

检查证书与私钥是否成对、DNS 名称、有效期、服务对文件的访问权限，以及所选 CA 或固定指纹策略。核心要求恰好选择一种信任方式：系统根证书、明确的 CA 文件或证书固定指纹。使用固定指纹时仍会检查主机名与有效期。保持证书验证开启，并按[安全](SECURITY.zh-CN.md)和 [Cloudflare](CLOUDFLARE.zh-CN.md) 设置正确的信任关系。

两台服务器的时间都必须正确。即使令牌正确，超过五分钟的时钟偏差也可能导致 WebSocket 预认证被拒绝。应检查现有时间同步服务，而不是反复生成新令牌。

## ECH 或 Fragment 失败

严格 ECH 同时要求 DNS HTTPS 记录中存在可用的 ECH 配置，以及服务器实际接受 ECH。`godoftun -check-ech DOMAIN` 只检查配置发现和可用性，不证明对端接受成功。Doctor 的 actual-carrier 阶段检查所选的真实握手。

严格 ECH 要求 TLS 1.3，不能与 ClientHello 分片同时启用。普通 TLS 探测成功，或某个浏览器配置名称，都不能证明 ECH 生效。对于 ECH/Fragment 配置，Doctor 会有意跳过普通 TLS 探测；这个 skipped 状态是预期行为。使用证书固定指纹策略时，也会跳过单独的 CA 验证探测。

诊断时，通过 `godoftunctl edit TUNNEL_ID` 每次修改一个设置，协调应用生成的对端更新，然后重新运行 Doctor。关闭 ECH 是显式配置变更，不是自动降级。Fragment 只改变第一个 ClientHello 的 TCP 写入边界，中间设备可以重新合并这些写入。二者均不保证连通性或更高速度。

## 断线、重启或各端口流量不均

对照相同时间段的两端隧道日志与 `health`。检查是否发生服务重启、远端会话关闭、配额耗尽或网络超时。CDN/前端重启和连接失败需要建立新会话；重新连接不会保留应用现有的 TCP 流。如果确实需要重启，先确认受影响隧道并接受其连接可能中断，再执行 `sudo godoftunctl restart TUNNEL_ID`。

多个境外端口按优先顺序配置，不代表每个端口承载相同流量，也不会自动聚合带宽。使用 `--endpoints` 逐一测试计划使用的境外端口。使用真实客户端测试每个伊朗入口端口，并检查对应客户的端口分配和配额。多个伊朗端口可以属于同一个客户，共享其配额。

发生资源压力时，请对照[配置](CONFIGURATION.zh-CN.md)中说明的保留缓冲区限制与进程实际内存。配置的内存预算不是整个进程的 RSS 上限。不检查内存和目标服务就增加会话/流数量限制，可能使问题加重。

## 配对更新被拒绝，或只更新了一端

在两端运行 `sudo godoftunctl edit-show TUNNEL_ID`，比较非敏感设置。当前编辑器生成的签名代码绑定特定配对和方向，七天后过期，并验证修改前的值。后续修改、错误隧道、错误对端、系统时间错误或 V3/3.x 代码，都可能触发 `Invalid, stale, expired, or wrong-peer update code`。

确认正确的配对并核对差异，然后从适当的服务器生成新的 V1 变更，通过 `sudo godoftunctl edit-apply TUNNEL_ID` 应用。`apply-update` 是同一操作的别名。不要手动修改编码后的代码、改变配对身份，或把机密信息贴到 issue 中。本地修改后，在对应对端更新完成前，流量可能中断；请遵循[运维指南](OPERATIONS.zh-CN.md)。

## 客户被阻止，或流量统计看起来不对

在伊朗端运行：

```bash
sudo godoftunctl usage TUNNEL_ID
sudo godoftunctl customers TUNNEL_ID
```

`disabled`、`exhausted`、`stopped`、`status stale`、`status unavailable` 和 `blocked` 表示不同情况。检查客户是否启用、客户的累计使用配额，以及隧道的总累计使用配额。上传和下载合并计量；`0` 表示该项配额不限量，1 GB 为 1,000,000,000 字节。修改配额会保留既有用量，因此新配额低于已记录用量时，不会解除限制。

管理器只在状态文件修改时间约五秒以内时，将其视为新鲜状态。服务停止、快照过期或快照缺失，不等于实时用量为零。`The service did not acknowledge the new customer configuration` 表示请求的配置版本尚未获得确认；重试前请检查所选服务及报告的计量错误。

持久用量账本位于 `/var/lib/godoftun/TUNNEL_ID/access-state.json`。`access state unavailable; refusing to reset usage`、校验和不匹配、状态文件权限不够私密或 `access state already in use` 等错误，需要检查对应配置/状态、文件属主和权限、存储以及重复进程。不要为了启动服务而删除或重建账本。保留备份，并使用[运维指南](OPERATIONS.zh-CN.md)中的恢复流程。

计量统计套接字上传/下载字节，不包括隧道帧和 padding。发生崩溃后，持久预留机制可能对每个活动账户保守地保留最多 64 KiB 用量；这些计数不一定等于提供商记录的网卡总流量。反复重置状态会破坏累计用量计量。

## 升级或回滚失败

保留程序打印的 `/opt/godoftun/backups` 下的恢复目录。阅读第一个升级错误及回滚结果。配置恢复成功并不证明网络路径正常；请再次检查服务状态、运行 Doctor，并进行真实客户端测试。

可执行文件和管理工具由多个隧道共享，但每个隧道的配置和服务独立。按[运维指南](OPERATIONS.zh-CN.md)升级两端服务器及每个受影响的隧道。不支持的配置或 3.x 安装可能阻止替换共享二进制文件；不要绕过此检查，也不要混用配对代码或版本。

如果回滚提示需要人工处理，请保留原始文件、失败后的文件及当前用量账本。不要把旧用量快照覆盖到较新的账本，也不要未经核对就恢复整台服务器的 Nginx 配置。自动回滚会尽可能恢复受管理的文件和服务状态；已下载的依赖、服务账户和已签发的证书可能保留。安装程序不修改 Xray。计量状态缺失时，程序有意拒绝将其视为零用量的新安装。

## Doctor 通过，但 Xray 客户端不能工作

按以下顺序检查剩余的应用路径：

1. 客户端连接伊朗 IP 和分配给该客户的端口，协议、凭据、传输和 TLS 设置与境外独立 Xray 入站一致。
2. 该客户已启用，客户配额与隧道总配额均允许继续传输。
3. 境外目标确实监听已配置的回环地址和端口，egress 目标允许列表包含这个准确目标。
4. Xray 入站接受所提供的客户端设置，其出站和 DNS 路径正常。
5. 真实客户端能上传和下载；在每个计划使用的入口端口测试短传输与持续传输。

服务器之间的 GODOFTUN WSS 连接不会自动改变 Xray 入站协议或 TLS 配置。保持独立入站与现有服务分离。拓扑见[安装](INSTALLATION.zh-CN.md)，端口用途见[配置](CONFIGURATION.zh-CN.md)。

## 提交脱敏支持报告

请提供两端 `godoftun -version`、操作系统/架构、隧道角色、失败阶段与 UTC 时间、相关端口用途、是否启用 ECH/Fragment/MUX、最近的相关变更，以及受影响隧道的短日志。附上 Doctor 的结论和各阶段结果，并说明真实客户端上传/下载是否成功。报告计量问题时，请提供显示的状态和快照时间差，而不是账本本身。

Doctor 会清除已知机密，但报告仍包含端点地址、SNI、隧道 ID 和诊断元数据。分享前请先检查。移除令牌、Authorization 头、私有 WebSocket 路径、配对/更新代码、私钥、客户导出文件或凭据、完整启动/配置文件，以及计量文件。不要附上整个备份目录。`--output` 不会覆盖已有文件；为新报告选择新的私有文件名。参见[安全](SECURITY.zh-CN.md)和本版本的[验证范围](../VALIDATION.md)。
