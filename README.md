# GODOFTUN V1.0.0

A multiport TCP tunnel for a Cloudflare-proxied foreign domain, with direct IP/port access on the Iran side.

[فارسی](README.fa.md) · [简体中文](README.zh-CN.md) · [Русский](README.ru.md) · [Community](https://t.me/SIKDPI)

## Documentation

Read installation and Cloudflare setup before running the installer. Examples use placeholders; replace them with your own server values.

| Guide | Contents |
|---|---|
| [Installation](docs/INSTALLATION.md) | Prerequisites, checksum verification, foreign/Iran setup, client settings and acceptance checklist |
| [Cloudflare](docs/CLOUDFLARE.md) | DNS, Proxy ON, ports, WebSockets, origin certificates, renewal and ECH |
| [Configuration](docs/CONFIGURATION.md) | Defaults, limits, MUX, padding, sessions, streams, memory, TLS and paired edits |
| [Operations](docs/OPERATIONS.md) | Command reference, customer ports, quotas, monitoring, transfer, backup, upgrade and removal |
| [Troubleshooting](docs/TROUBLESHOOTING.md) | Doctor, common failures, recovery and safe support reports |
| [Security](docs/SECURITY.md) | Visible code, secret handling, trust boundaries and distribution limits |

All six guides are available in the four languages above. Commands and the installed terminal interface remain in English. [Validation results](VALIDATION.md) distinguish completed local tests from unexecuted Linux and real Cloudflare tests.

## Download and run

Extract the complete public ZIP and enter its `sik-dpi` folder. Use the standalone `GODOFTUN-V1.0.0.sh` from the same trusted release. On Linux, verify all files before use:

```bash
sha256sum -c SHA256SUMS
```

On macOS, use `shasum -a 256 -c SHA256SUMS`. A checksum does not authenticate an attacker-controlled file and manifest together.

Preview the menu without installing:

```bash
bash GODOFTUN-V1.0.0.sh preview
```

On a clean test server, set up the foreign side first:

```bash
sudo bash GODOFTUN-V1.0.0.sh foreign
```

Then run the installer on Iran and paste the confidential pairing code:

```bash
sudo bash GODOFTUN-V1.0.0.sh iran
```

The installer embeds Linux amd64 and arm64 executables. Go is not required on the target servers. Ubuntu/Debian with systemd and public IPv4 is the intended installation environment. Dependencies such as Nginx may still require package downloads. A copied installer is not a fully offline deployment: DNS, Cloudflare proxy verification and the tunnel path must be reachable. This is TCP only, not a UDP or whole-device VPN.

## Features

- Foreign domain with Cloudflare Proxy ON; WSS only.
- Multiple client ports and named tunnels.
- MUX on/off, bounded sessions, stream and retained-buffer limits.
- Byte padding, default 0.22; strict ECH or ClientHello fragmentation.
- Paired configuration editing, customer port quotas and diagnostics.
- Teal terminal menus grouped into Tunnels, Tuning, Customers and Doctor; step-by-step setup.

Arvan, a direct foreign-domain mode, and an Iran-side domain are not offered. V1 keeps its fixed 512 KiB flow window; the 3.5 adaptive window controller is not included. Do not mix V1 and 3.x installations.

## Terminal interface

Run the installer without an argument for the main menu, or use `manage` after installation. The `preview` command displays menus without root, network checks or server changes. It shows no simulated live status.

Colors adapt to dark/light terminals with `GODOFTUN_THEME=dark` (default) or `GODOFTUN_THEME=light`. Use `NO_COLOR=1` for plain text; file output and `TERM=dumb` are plain automatically. Menus wrap for narrow terminals; `COLUMNS=60` sets a 60-column layout. Existing action numbers and confirmations are retained.

## Source and safety

This is a **binary distribution**, not an open-source repository. The Go implementation is not included. The installer and embedded shell/Python management code are readable; compiled executables can also be analyzed. No claim of complete source concealment is made.

Read [validation results](VALIDATION.md) and complete real Linux installation and Cloudflare-path tests on your own clean servers before deploying to users. Do not share tokens, private keys, pairing/update codes or customer accounting files.

Project distribution terms are pending the owner's approval. Dependency notices are included in `THIRD_PARTY_LICENSES/`. See [acknowledgments](ACKNOWLEDGMENTS.md).
