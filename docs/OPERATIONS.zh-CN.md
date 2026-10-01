# 运维 — GODOFTUN V1.0.0

[English](OPERATIONS.md) · [فارسی](OPERATIONS.fa.md) · [简体中文](OPERATIONS.zh-CN.md) · [Русский](OPERATIONS.ru.md)

[首页](../README.zh-CN.md) · [安装](INSTALLATION.zh-CN.md) · [配置](CONFIGURATION.zh-CN.md) · [Cloudflare](CLOUDFLARE.zh-CN.md) · [故障排查](TROUBLESHOOTING.zh-CN.md) · [安全](SECURITY.zh-CN.md)

以下命令是供你在自己的服务器上执行的说明，不表示某个实际部署已经通过测试。将 `TUNNEL_ID` 和需要时的 `CUSTOMER_ID` 替换为真实 ID。管理命令需要 root 权限。首次部署前阅读[安装](INSTALLATION.zh-CN.md)，修改共享设置或配额前阅读[配置](CONFIGURATION.zh-CN.md)。

## 找到正确的隧道

```bash
sudo godoftunctl list
sudo godoftunctl status TUNNEL_ID
sudo godoftunctl health TUNNEL_ID
sudo godoftunctl edit-show TUNNEL_ID
```

显示名称不是内部 ID。本地服务名为 `godoftun@TUNNEL_ID`，配置位于 `/etc/godoftun/tunnels/TUNNEL_ID`，计量状态位于 `/var/lib/godoftun/TUNNEL_ID`。在自己的脚本中请明确写出 ID。如果省略，管理器会选择唯一的隧道，或在存在多个隧道时要求选择。

## 安装器命令

| `bash ./GODOFTUN-V1.0.0.sh` 的参数 | 操作 |
|---|---|
| 无参数 | 主菜单：境外安装、伊朗安装、管理、传输指南或 V1 升级 |
| `preview` 或 `--preview` | 显示菜单；不需要 root，不进行网络检查、安装，也不伪造实时状态 |
| `foreign` 或 `egress` | 安装境外侧；需要 root |
| `iran` 或 `origin` | 安装伊朗侧；需要 root 和境外配对码 |
| `manage` | 管理已有且受支持的 V1 隧道；需要 root |
| `upgrade` | 升级选中的受支持 V1 安装；需要 root |
| `transfer-help` | 显示文件传输说明 |

已安装的副本位于 `/opt/godoftun/share/godoftun-easy.sh`。升级时应使用新下载并验证的发行版安装器；重新运行旧副本不会下载新版。

## 完整的 `godoftunctl` 命令表

除特别说明外，格式为 `sudo godoftunctl COMMAND TUNNEL_ID`。`list` 不需要 ID，`help` 显示用法。只显示信息的命令不能证明真实客户端流量可用。

| 命令 | 用途 |
|---|---|
| `list` | 列出当前服务器的隧道 |
| `status` | 详细 systemd 服务状态 |
| `health` | 服务状态、PID/重启次数、进程资源、当前这次运行的统计和监听器 |
| `monitor` | 每两秒刷新健康状态；Ctrl+C 退出 |
| `logs` | 显示最近 100 条 journal 记录并持续跟踪；Ctrl+C 退出 |
| `start`、`stop`、`restart` | 只改变所选隧道服务的运行状态 |
| `config` | 显示受管理的启动命令；正常安装使用 token 文件路径而非内联 token |
| `info` | 客户端端点和入站说明 |
| `ports` | 已登记端口，包括内部基础设施预留，不只是公网客户端端口 |
| `peer-code` | 在境外侧显示机密安装配对码 |
| `test` | 检查服务及配置的 TLS 路径；不是端到端应用测试 |
| `doctor`、`scan` | 同一个有范围限制的诊断工具，参数见下文 |
| `edit` | 打开完整设置编辑器 |
| `edit-show` | 显示可编辑设置，隐藏 token |
| `edit-ip` | 询问要修改伊朗还是境外 IP，并执行编辑 |
| `edit-ports` | 境外修改 HTTPS 端口；伊朗修改默认接入端口 |
| `edit-apply`、`apply-update` | 应用机密、经过签名认证的 V1 编辑更新；两者为别名 |
| `customers` | 在伊朗打开客户管理 |
| `usage` | 显示客户配额、记录用量、余额和状态 |
| `customer-add` | 添加独立客户入口并生成端点说明 |
| `customer-quota` | 修改单个客户的累计上限，不清零用量 |
| `customer-disable`、`customer-enable` | 改变访问状态，保留账户及用量 |
| `customer-export` | 写入并显示客户端点 JSON 文件 |
| `customer-ports` | 修改单个客户的直连接入端口，保留用量 |
| `tunnel-quota` | 修改整个隧道的累计共享上限 |
| `delete` | 输入准确 ID 后删除一个隧道；参见删除章节 |
| `help` | 显示命令用法 |

针对现有客户的命令可将 `CUSTOMER_ID` 作为第三个位置参数，否则会要求选择。端口列表和配额通过交互提示输入，不是额外的未记录 CLI 参数。例如：

```bash
sudo godoftunctl customer-export TUNNEL_ID CUSTOMER_ID
sudo godoftunctl customer-quota TUNNEL_ID CUSTOMER_ID
sudo godoftunctl customer-ports TUNNEL_ID CUSTOMER_ID
```

配额/端口操作会在输入完成后保存；不要假定每项客户操作都有编辑器那样独立的 `Apply?` 确认。没有 `customer-delete`、按月续期或用量清零命令。分配额度前请阅读[配置中的客户计量说明](CONFIGURATION.zh-CN.md)。

## 监控与有限范围诊断

先使用 `status` 和 `health`。Health 显示的是当前这次服务运行的统计，不是之前运行留下的会话。系统工具显示的进程 RSS 与保留缓冲区预算不同。`usage` 会标记过期/已停止的快照；旧快照不是实时读数。服务为 `active` 并不证明 Xray 认证、载荷传输或持续速度正常。

以下检查不会修改隧道配置。第一条是离线检查；其他命令会访问配置中的端点或明确指定的端点：

```bash
sudo godoftunctl doctor TUNNEL_ID --offline
sudo godoftunctl doctor TUNNEL_ID --timeout 8
sudo godoftunctl doctor TUNNEL_ID --endpoints tunnel.example.com:443,tunnel.example.com:8443
sudo godoftunctl doctor TUNNEL_ID --output /root/godoftun-doctor.json
```

`tunnel.example.com` 是文档示例，只能替换为你拥有或获准测试的端点。`doctor`/`scan` 最多接受 16 个明确的主机/IP 候选项，不接受地址范围。`--timeout` 为 1–20 秒，默认 8 秒，限制每个阶段及核心探测的总时长。`--binary` 是指定匹配可执行文件的高级覆盖项，默认 `/usr/local/bin/godoftun`，不能指向 `run.sh` 启动脚本。

`--offline` 不发送网络请求，只检查本地配置，不能认证连通性。在线成功要求一次新的、经过认证的承载连接检查；严格 ECH 使用核心实际执行 ECH 握手。即使结果为 `carrier-authenticated`，也没有测试最终应用载荷或客户凭据。退出码 0 表示承载连接认证成功或离线配置检查成功；1 表示诊断结果不成功；2 表示配置/报告错误。

`--output` 创建私有、已脱敏的 JSON 文件，拒绝覆盖现有文件或符号链接。再次运行时请使用新文件名。分享前仍应检查报告。原始日志、`config`、`info`、`edit-show` 和截图可能暴露地址、路径或运维细节。`peer-code`、配对文件、编辑码和备份都是机密信息，切勿贴到公开 issue。参见[安全](SECURITY.zh-CN.md)与[故障排查](TROUBLESHOOTING.zh-CN.md)。

## 同步两端修改

在一端使用 `edit`，在另一端使用 `edit-apply`。安全传递生成的代码，并在交互提示中粘贴，不要把它放进 shell 命令或公开消息。在下一项共享修改前，先完成上一项的对端同步。两台服务器的本地隧道 ID 可能不同；应匹配实际配对关系，而不仅是相似的显示名称。

编辑码七天后失效，并检查旧值、配对身份和相反角色。`apply-update` 现在调用同一签名编辑器，不能用于旧原始更新机制。`peer-code` 用于初次安装，不用于应用编辑。修改域名/IP 后，DNS 和客户设备配置仍需自行修改；已经发给客户的端点文件不会远程自动更新。详见[配置](CONFIGURATION.zh-CN.md)。

## 下载受限时传输安装器

完整安装器包含两种 Linux 架构。请从可信设备复制已验证的文件；仅接收可执行文件时，伊朗服务器不需要 Go 或 GitHub 访问权限。以下 IP 均为文档保留地址，必须替换为自己的服务器地址。

从可信设备将发行文件推送到伊朗：

```bash
scp ./GODOFTUN-V1.0.0.sh root@203.0.113.20:/root/GODOFTUN-V1.0.0.sh
```

也可以在伊朗服务器执行以下命令，从境外服务器拉取已安装的副本：

```bash
scp root@198.51.100.10:/opt/godoftun/share/godoftun-easy.sh /root/GODOFTUN-V1.0.0.sh
```

只有该副本确实是需要的 V1.0.0 安装器时，才应使用这里的文件名。确认 SSH 主机密钥，比较收到文件的 SHA-256 与可信原件一致后再执行：

```bash
sha256sum /root/GODOFTUN-V1.0.0.sh
sudo bash /root/GODOFTUN-V1.0.0.sh iran
```

这只是离线**交付文件**，不保证整个安装过程离线：系统软件包、DNS、证书检查和实际 Cloudflare 路径仍可能需要网络，所需依赖也必须可用。安装器可使用 `/etc/godoftun/tunnels/TUNNEL_ID/ca-bundle.pem` 或共享路径 `/etc/godoftun/ca-bundle.pem` 中的有效 PEM CA 包；否则使用系统 CA 包，或尝试安装 `ca-certificates`。只从可信来源传输公开 CA 证书，不要传输服务器私钥。参见[安装](INSTALLATION.zh-CN.md)。

## 升级已有 V1 隧道

升级只面向已登记的 `GODOFTUN_V1_CF_WSS` 安装，不是从 3.x 迁移。现有隧道元数据不受支持、境外连接模式不兼容、或已存在未登记的可执行文件时，共享二进制替换会被阻止。不要删除这些检查，也不要将 3.x 安装重新标记为 V1。

1. 阅读新版本的验证说明，保存之前已验证的安装器及安全备份，为两端安排维护窗口。
2. 在各服务器上用新安装器的 `upgrade` 选择正确的本地隧道。按提示检查 Cloudflare DNS，并仔细确认。
3. 对两台服务器上的每个已有隧道重复操作。可执行文件和管理工具是共享的，但每次仅更新所选隧道的配置和服务。其他服务不会自动重启；它们下次重启时会使用共享的新可执行文件。
4. 检查 `status`、`test`、`doctor`，再进行真实客户端上传/下载、多端口、配额和重连测试。服务检查本身不能证明网络路径正常。

```bash
sudo bash ./GODOFTUN-V1.0.0.sh upgrade
```

现有受支持配置、证书、ID、配额和用量会保留。已停止的隧道保持停止。如果计量配置存在而账本丢失，升级会停止，不会给予新的用量额度。受支持的旧 V1 若从未启用计量，则会询问初始总额度；计量从本次升级开始，因为历史流量无法重建。

## 回滚、备份和计量

安装/升级事务、编辑器修改和客户修改会在 `/opt/godoftun/backups/` 下保存与本次操作范围对应的恢复副本。每项操作都会打印自己的备份位置。操作失败时会尝试恢复它修改的文件和服务状态；若无法确认安全恢复，受影响服务可能停止，并要求人工处理。

这些不是可以互换的完整系统快照。编辑器/客户回滚刻意不恢复旧的实时流量账本，以免退还已经消耗的字节。不要把旧 `access-state.json` 覆盖到正在运行或更新的账本上。账本损坏、状态缺失或配置确认失败必须排查；为了让服务启动而删除计量文件，会破坏用量保留保证。

没有受支持的 `godoftunctl restore` 或一键降级命令。应按自己的备份流程，保护好相关配置、当前计量状态、匹配的发行版和证书资料。保持一致的手工恢复可能需要停止受影响隧道并协调两端。应根据该操作实际生成的备份和错误报告恢复，而非使用猜测的通用恢复命令。备份含有秘密，必须私下保存。

## 停止服务、停用客户或删除隧道

这三种操作不同：

- `stop` 停止一个隧道服务，但保留安装、设置和用量；`start` 可重新启动。
- `customer-disable` 停用账户并关闭访问，但保留身份、计数器、配额和已登记端口。`customer-enable` 不会清零用量，也不会绕过已用尽的配额。
- `delete` 将整个所选隧道从管理中移除，不是删除客户，也不是卸载整个程序。

请先确认 ID，只有确实要移除该隧道时才运行：

```bash
sudo godoftunctl delete TUNNEL_ID
```

管理器要求输入准确的隧道 ID。它停止并禁用该服务，将配置和状态移入私有 `deleted-TUNNEL_ID-...` 备份；仅在没有其他隧道引用时，才移除其受管理的 Nginx 站点。会检查 Nginx 配置和重载结果；失败时尝试回滚。

共享可执行文件、Nginx 本身、其他隧道、证书和防火墙规则均保留。请自行审查闲置的防火墙开放端口；不要认为删除一个隧道就能安全关闭所有端口。在一台服务器删除不会删除对端、DNS 记录或客户端设备。保留的备份是可恢复数据，不是自动恢复功能。保留凭据的处理见[安全](SECURITY.zh-CN.md)，删除或恢复报错见[故障排查](TROUBLESHOOTING.zh-CN.md)。
