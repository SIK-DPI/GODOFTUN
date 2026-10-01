# Install GODOFTUN V1.0.0

[فارسی](INSTALLATION.fa.md) · [简体中文](INSTALLATION.zh-CN.md) · [Русский](INSTALLATION.ru.md) · [Home](../README.md)

This guide installs one paired tunnel on two servers. Set up the foreign server first, then Iran. Test on separate servers before replacing a working service.

## What connects to what

```text
Client application
  -> Iran public IP and TCP entry port
  -> GODOFTUN origin
  -> foreign domain through Cloudflare WSS
  -> foreign Nginx and GODOFTUN egress
  -> dedicated application inbound on 127.0.0.1
```

The supported profile is fixed: foreign Cloudflare Proxy ON, WSS, direct Iran IP entry. There is no Iran-domain, Arvan, direct-foreign, HTTP/2 or XHTTP choice. This is a TCP tunnel, not a UDP or whole-device VPN. An Xray/panel application still needs its own protocol, credentials and client configuration. GODOFTUN is not the Cloudflare Tunnel or `cloudflared` product.

## Before starting

- Two Ubuntu or Debian servers with systemd, root access through `sudo`, and public IPv4 addresses. The installer accepts amd64/x86_64 and arm64/aarch64, not 32-bit systems. macOS is not an installation target.
- A separate foreign subdomain you control, such as `tunnel.example.com`, configured using [the Cloudflare guide](CLOUDFLARE.md). This example name is not a working endpoint; replace it with yours.
- A dedicated application inbound on the foreign server, listening on `127.0.0.1` at an unused TCP port. For example, `18082` only if you have actually created that inbound there. Do not change an unrelated panel inbound.
- An origin certificate and key, or access for the installer's Let's Encrypt HTTP challenge. An edge certificate alone is insufficient.
- Access to package repositories for missing tools, Cloudflare's IP-range API for proxy verification, and the intended foreign HTTPS endpoint. ECH additionally needs working HTTPS DNS discovery. Copying the installer offline does not remove these network requirements.
- Synchronized server clocks and existing SSH access protected from firewall changes.

The installer checks the OS family; it does not certify every Ubuntu/Debian release. See [tested scope](../VALIDATION.md). Use supported, updated OS packages. No Go compiler is required on either server.

## Verify the download

Extract the entire public ZIP, enter its `sik-dpi` folder and keep `SHA256SUMS` with the files. Run on Linux:

```bash
sha256sum -c SHA256SUMS
bash -n GODOFTUN-V1.0.0.sh
bash GODOFTUN-V1.0.0.sh preview
```

On macOS, use `shasum -a 256 -c SHA256SUMS` for the first command; the menu preview also works without installation. Every listed file must be present for a complete checksum check. A checksum detects changes against that manifest, not the authenticity of a malicious replacement manifest. Get both from the owner's trusted release.

The `preview` command only displays menus. Running without an argument opens the interactive main menu. Do not run an unverified download directly through a privileged shell.

## Network ports

| Location | Required access |
|---|---|
| Foreign public interface | Selected HTTPS TCP ports, normally 443; TCP 80 for the installed HTTP site and automatic certificate challenge |
| Foreign loopback | Dedicated application inbound and the private WSS backend selected by the installer; do not open these publicly |
| Iran public interface | The client TCP entry ports you choose, for example 21000 and 21001 if free |
| Iran outbound | DNS, HTTPS for prerequisite checks, and the foreign domain's selected HTTPS ports |
| Both servers | Your existing SSH access and necessary package/time services |

The public HTTPS ports and Iran client ports are different settings; they need not match. Multiple Iran ports in one tunnel reach the same configured foreign application target. Use another named tunnel, with separate resources, for another target. On an already active UFW firewall the installer adds required TCP rules; it does not enable UFW or configure your provider's firewall. Review both. Do not flush a shared firewall.

## Set up the foreign server

First create the application inbound and finish the [DNS and certificate preparation](CLOUDFLARE.md). Then run:

```bash
sudo bash GODOFTUN-V1.0.0.sh foreign
```

Answer the prompts:

1. Choose a tunnel ID and friendly name. IDs use 1–48 lowercase letters, digits and single hyphens, begin with a letter or digit, and cannot contain consecutive hyphens. Use a new ID for a new tunnel.
2. Choose MUX on or off. Start with the default unless you have a specific reason; see [configuration](CONFIGURATION.md).
3. Enter the foreign domain with the orange cloud enabled and the foreign server's real IPv4 address. The public DNS answer should be Cloudflare, not that origin address.
4. Enter HTTPS ports in priority order. Start with `443`. Only the listed Cloudflare-compatible HTTPS ports are accepted.
5. Enter the dedicated inbound's actual port. The installer checks `127.0.0.1:PORT` and chooses a separate free private WSS backend.
6. Choose the browser TLS profile and padding intensity, default `0.22`.
7. Select automatic Let's Encrypt or an existing certificate/key and local trust bundle. Resolve preflight errors before confirming installation.

After startup, local HTTPS, an HMAC-authenticated WebSocket upgrade (not the full V1 challenge-response) and the CDN path are checked. Keep the generated pairing code private: it contains connection details and a shared token. Base64 is not encryption. Do not put the code in a public issue, screenshot or GitHub file.

## Set up Iran

Transfer the same checked installer to Iran; [operations](OPERATIONS.md) explains transfer when GitHub is unreachable. Run:

```bash
sudo bash GODOFTUN-V1.0.0.sh iran
```

1. Answer whether another tunnel is already registered. This does not authorize replacing it.
2. Paste the foreign pairing code. Leaving it blank opens manual entry; use the code for a first installation to avoid mismatches.
3. Choose the local tunnel ID/name when prompted, then enter Iran's public IPv4 address. No Iran domain is requested.
4. Enter one or more unused client ports separated by commas, such as `21000,21001`.
5. Choose the session limit and padding. MUX and transport are imported from the paired endpoint.
6. Choose ECH OFF, strict ECH ON, or ECH OFF with experimental ClientHello Fragment. ECH and Fragment are alternatives, not a combined mode. See [configuration](CONFIGURATION.md).
7. Set the total lifetime allowance in decimal GB. `0` means unlimited, with upload and download counted together.
8. Confirm after preflight passes. The installer validates transport sessions; it does not validate your application's credentials or payload.

If input or validation fails, read the error and retry the indicated step. Do not bypass checks by turning off certificate verification or changing to an unsupported transport.

## Connect and test a real client

On each server, find its tunnel ID:

```bash
sudo godoftunctl list
sudo godoftunctl status TUNNEL_ID
sudo godoftunctl doctor TUNNEL_ID
```

Replace `TUNNEL_ID` with the listed local ID. On Iran, `sudo godoftunctl info TUNNEL_ID` displays client guidance. Review this output privately. Set the client's connection address to the Iran IP and its assigned entry port; retain the dedicated foreign inbound's protocol, UUID/password and application-layer security settings. If that application uses TLS, its SNI/certificate requirements still apply. Do not confuse application TLS with the tunnel's outer WSS TLS.

Before serving users, verify all of the following on your actual route:

- A real authenticated client uploads and downloads through every intended Iran port.
- Concurrent connections and a larger transfer work without truncated responses.
- A planned service restart allows new connections to recover; established streams are not promised seamless survival.
- Strict ECH is accepted when selected; Fragment is tested separately, not assumed to help.
- A limited test customer reaches its quota and is blocked correctly; raising a quota does not erase recorded usage.
- Diagnostics, certificate renewal and access after a server reboot work. A process marked `active` or a generic web page is not enough.

Keep test results private and redact credentials before asking for help. Continue with [operations](OPERATIONS.md), [troubleshooting](TROUBLESHOOTING.md) and [security](SECURITY.md).
