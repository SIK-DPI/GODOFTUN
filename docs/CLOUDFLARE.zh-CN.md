# 配置境外 Cloudflare 域名

[English](CLOUDFLARE.md) · [فارسی](CLOUDFLARE.fa.md) · [Русский](CLOUDFLARE.ru.md) · [首页](../README.zh-CN.md)

请为 GODOFTUN V1.0.0 使用专用子域名。这些设置需要在你的 Cloudflare 账户中完成；安装器不会创建 DNS 记录，也不会修改账户设置。控制面板中的标签或位置可能变化。本文所链接的官方文档已于 2026-10-01 核对。

## DNS

在该域的 DNS 记录中，为所选子域名创建一条 A 记录，指向境外服务器的公网 IPv4 地址。将代理状态设为 Proxied，即橙色云朵。开启代理后，DNS 返回的是 Cloudflare 地址，而不是服务器地址。[代理状态](https://developers.cloudflare.com/dns/proxy-status/)

首次部署时，只使用一个预期的源站。不要为该名称添加无关的 AAAA 记录、负载均衡器或 Worker 路由。修改之前，请检查已有记录。伊朗端不需要 DNS 记录。

在两台服务器上，将示例域名替换为自己的域名，并检查解析结果：

```bash
getent ahostsv4 tunnel.example.com
getent ahostsv6 tunnel.example.com
```

GODOFTUN 会将解析到的地址与 `https://api.cloudflare.com/client/v4/ips` 提供的 Cloudflare 官方地址范围进行比对。混合解析、直接指向源站、非 Cloudflare 地址或无法验证的结果都会被拒绝。如果无法访问 API，请先恢复访问再重试；此版本没有绕过该检查的离线选项。

## HTTPS 端口和 WebSockets

先使用 TCP 443。GODOFTUN 接受 HTTPS 端口 `443,2053,2083,2087,2096,8443`，与 Cloudflare 支持的代理端口一致。[网络端口](https://developers.cloudflare.com/fundamentals/reference/network-ports/)

在 Network 设置中启用 WebSockets。初始协议升级可能受到安全规则影响，Cloudflare 也可能关闭长时间保持的连接。[WebSockets](https://developers.cloudflare.com/network/websockets/)

在此部署中，需要保留域名的 Host/SNI 和私有路径。不要对经过身份验证的 WSS 请求应用重定向、缓存、交互式浏览器验证或请求头改写。如果某条安全规则阻止了请求，请调查具体事件，并仅针对隧道主机名/路径设置范围最小且有充分理由的例外。不要关闭整个域的防护。GODOFTUN 会生成 Nginx 协议升级路由；手动替换该路由可能破坏身份验证。

## 两项独立的证书检查

伊朗连接端会验证公网边缘证书。境外服务器上的 Nginx 则需要单独的源站证书。在 SSL/TLS 设置中选择 Full (strict)，并使用有效期内、覆盖该主机名且由公开受信任的 CA 或 Cloudflare Origin CA 签发的源站证书。仅通过本地 CA 证书包的检查，并不意味着 Cloudflare 信任该证书。[Full strict 模式](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/full-strict/)

安装器提供两种源站证书方式：

| 方式 | 需要提供的内容 |
|---|---|
| 自动申请 Let's Encrypt | 电子邮件地址和可访问的 HTTP 验证路径；安装器通过 Certbot 申请证书 |
| 现有证书 | 不含空格的证书链及未加密私钥的绝对路径，以及本地源站验证使用的 CA 证书包 |

本地检查会验证主机名、证书与私钥是否匹配，以及证书有效期；提供的证书如果剩余有效期少于 24 小时，会被拒绝。不要把私钥内容粘贴到终端提示中：应输入私密存储文件的路径。

对于 Cloudflare Origin CA 证书，本地源站检查需要使用对应的 CA 根证书包。Origin CA 不属于普通浏览器的信任体系。[Origin CA](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca/)

伊朗连接端仍使用公网边缘的信任库，而不是你的 Origin CA 证书包。编辑器提供独立的本地源站 CA 和边缘 CA 字段。不要通过关闭验证来解决证书错误。

## 首次签发与续期

自动签发使用境外服务器 HTTP webroot 下的 `/.well-known/acme-challenge/`。必须能够访问 TCP 80。请确保此路径到达预期源站，不受到交互式验证、缓存的影响，也不会被重定向到尚未建立的 HTTPS 监听器。保持 Proxy ON。任何临时的路由/重定向例外都应仅限于验证路径，并在完成安装前确认正常的 Full (strict) HTTPS 已经可用。

安装器可以创建 Certbot 续期支持和受管理的 Nginx 重载钩子。现有的非 Certbot 证书仍由你自行负责。请监测到期时间，并在 Linux 验收测试中验证续期安排。不要认为安装时成功就能证明今后的续期也会成功。

如果账户规则无法在不削弱无关服务保护的前提下完成 HTTP 验证初始化，请选择已经签发的证书。安装器不提供 DNS API 验证流程，也不会要求你的 Cloudflare API 令牌。

## ECH

Cloudflare 在 SSL/TLS 边缘证书文档中说明了 ECH。可用性和设置取决于具体域；ECH 会向中间方隐藏内部 ClientHello 的主机名，但不会向 Cloudflare 隐藏。[ECH 配置](https://developers.cloudflare.com/ssl/edge-certificates/ech/)

GODOFTUN 的严格 ECH 模式需要 HTTPS 记录公布 ECH 配置、能够完成发现查询，并且 TLS 1.3 握手实际接受该配置。仅在控制面板中打开开关并不能证明这些条件已经满足。请使用实际 GODOFTUN 连接和 Doctor 结果进行确认；访问网站或使用普通 TLS 检查不能复现其握手。请参阅 [ECH 和 Fragment 设置](CONFIGURATION.zh-CN.md)。

## 确认完整路径

境外端安装完成后，对域名根路径发出的普通 HTTPS 请求应到达生成的通用站点。其中的 “All systems operational” 是静态伪装页面文本，并非实时健康检查结果。然后按照[安装指南](INSTALLATION.zh-CN.md)完成伊朗端安装和真实应用传输测试。

请保护源站地址和应用目标。Cloudflare 会终止 WSS 这一段的 TLS 连接，因此该方案并不提供对 Cloudflare 的端到端保密；如果需要端到端加密，应由隧道内的应用提供。分发客户端接入信息之前，请阅读[安全指南](SECURITY.zh-CN.md)。
