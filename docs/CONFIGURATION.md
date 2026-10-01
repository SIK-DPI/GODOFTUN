# Configuration — GODOFTUN V1.0.0

[English](CONFIGURATION.md) · [فارسی](CONFIGURATION.fa.md) · [简体中文](CONFIGURATION.zh-CN.md) · [Русский](CONFIGURATION.ru.md)

[Home](../README.md) · [Installation](INSTALLATION.md) · [Cloudflare](CLOUDFLARE.md) · [Operations](OPERATIONS.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Security](SECURITY.md)

GODOFTUN V1.0.0 has one deployment profile: a Cloudflare-proxied foreign domain over WSS, with direct IP/port entry on Iran. The editor cannot switch to Arvan, foreign Proxy OFF, an Iran domain, HTTP/2 or XHTTP. It does not configure your Xray clients, inbound authentication, Cloudflare account or DNS records.

## Open the settings editor

Replace `TUNNEL_ID` with the ID shown by `list`. Run these commands on the server whose settings you want to inspect or change:

```bash
sudo godoftunctl list
sudo godoftunctl edit-show TUNNEL_ID
sudo godoftunctl edit TUNNEL_ID
```

`edit-show` displays configuration, not live connection health. The token is hidden. The editor previews the old and new values and asks `Apply? y/n`, defaulting to `n`. An active affected service may restart; a stopped service stays stopped. Use the menu instead of rewriting `run.sh` or metadata by hand: the managed editor checks related limits, ports and peer settings together.

## Connection and identity

| Setting | Meaning and limits | Scope |
|---|---|---|
| Tunnel display name | Printable text, up to 120 UTF-8 bytes, without `=`. Does not rename the internal tunnel ID or pairing identity. | Local |
| Foreign server IPv4 | Origin-server address used for metadata and checks; the Iran connector still dials the proxied domain. Update the actual DNS origin yourself when moving the server. | Shared |
| Iran server IPv4 | Public address distributed to clients. Existing client devices do not update themselves. | Shared |
| Foreign domain | A DNS hostname such as `tunnel.example.com`, without a scheme, port or path. All resolved addresses must pass the editor's Cloudflare proxy check. | Shared |
| Foreign HTTPS ports | Distinct comma-separated ports from `443,2053,2083,2087,2096,8443`; installation suggests `443`. | Shared |
| Iran entry ports | Direct TCP ports for the default customer entry. Other customer accounts have their own port settings. Occupied or reserved ports are rejected. | Shared metadata; listeners change on Iran |
| Tunnel WebSocket path | `/` followed by 15–511 letters, digits, `/`, `_` or `-`. Generated during installation; both peers must use the same path. | Shared |
| Foreign Xray inbound port | An existing dedicated TCP target on `127.0.0.1` at the foreign server. The editor checks conflicts; port 80 is reserved for certificate challenges. | Shared |
| Rotate shared tunnel secret | Generates a new random token without displaying the old token. Coordinate the peer update immediately. | Shared |
| Certificate and private-key files | Existing absolute file paths for the managed foreign Nginx site. The certificate must cover the configured domain, be valid and match the key. Iran has no public TLS site in this edition. | Local to the foreign server |

Port numbers are 1–65535, subject to Cloudflare, ownership and listener checks. Do not reuse a tunnel's private backend or target port as a public entry. Changing the foreign domain requires matching certificate files on the foreign server; the editor does not silently issue a replacement or rewrite unrelated Nginx sites.

## MUX, padding and resource limits

| Setting | Default in a new installation | Allowed by the editor |
|---|---|---|
| MUX | `on` | `on` or `off`; shared edit |
| Padding intensity | `0.22`, carried in the pairing code | Finite number from `0` to `1`; shared edit |
| Minimum sessions | `2`; `1` when maximum sessions is `1` | `1`–`64`, no greater than maximum; Iran only |
| Maximum sessions | `8` with MUX on; suggested `32` with MUX off | `1`–`64`; Iran only |
| Maximum streams | Selected from the RAM profile below | `1`–`65,536`; local |
| Maximum streams per session | `512` | `1`–`512`; local |
| Global retained-buffer memory | Selected from the RAM profile below | `16`–`1,048,576` MiB; local |
| Retained memory per stream | `2,048` KiB | `64`–`1,048,576` KiB; local |
| Tunnel write timeout | `10s` | Whole seconds `1s`–`60s`; local |
| Client/target write timeout | `0` | `0` disables it; otherwise whole seconds `1s`–`300s`; local |

The installer selects limits independently on each server from `/proc/meminfo`:

| Reported physical RAM | Global retained-buffer budget | Maximum streams |
|---|---:|---:|
| Below 1.5 GiB | 256 MiB | 1,024 |
| At least 1.5 GiB, below 4 GiB | 512 MiB | 2,048 |
| At least 4 GiB, or RAM not detected | 1,024 MiB | 4,096 |

These are installation defaults, not recommended capacity figures. Inspect the saved values with `edit-show`. The standalone core's fallback is 512 MiB and 512 streams; the installer overrides those two values with the RAM profile.

The global memory budget must cover both the per-stream budget and at least 64 KiB for every configured stream. Raising the stream count may require raising memory first; lowering memory may require lowering stream limits first. These budgets count retained receive buffers and in-progress writes, **not total process RSS**. Other allocations still consume RAM.

With MUX on, several client connections share tunnel sessions. The pool starts at the minimum and can grow to the maximum under load. With MUX off, each client connection needs a dedicated session; the maximum session setting limits concurrent dedicated carriers, and minimum pooling is ignored. MUX off generally needs more resources.

Padding intensity is not a percentage of bandwidth or payload length: `0.22` does not mean 22% extra traffic. Activation and byte count are independent of payload length. `0` disables padding; data padding is suppressed under queue pressure. Higher settings can add overhead without improving your particular route. This release retains V1's fixed 512 KiB flow-control window; it has no 3.5 adaptive RTT/loss window controller.

## TLS, ECH and Fragment

These connection settings are edited on Iran, where the outgoing TLS connection is made:

| Setting | Default | Choices |
|---|---|---|
| TLS fingerprint profile | `chrome` | `chrome`, `firefox`, `ios` |
| Minimum TLS | `1.3` | `1.3`, or explicit `1.2` compatibility mode |
| ECH | `off` | `off`, `strict` |
| ClientHello fragmentation | `off` | `off`, `hello` |
| Fragment size | `100-200` bytes | `MIN-MAX`, with `16 ≤ MIN ≤ MAX ≤ 4096` |
| Delay between fragments | `1-3ms` | `MIN-MAXms`, with `0 ≤ MIN ≤ MAX ≤ 20` |
| Trusted CA bundle | Selected at installation | Existing absolute PEM-bundle path without spaces |

The installation menu offers ECH off, strict ECH, or ECH off plus experimental Fragment. Strict ECH requires TLS 1.3 and Fragment off. Turn the conflicting mode off before enabling the other. A strict connection fails if ECH discovery fails or the server does not accept ECH; there is no silent fallback to ordinary TLS.

ECH discovery queries HTTPS DNS records through the following DoH endpoints, in order:

1. `https://cloudflare-dns.com/dns-query`
2. `https://dns.google/dns-query`

The core's advanced `-ech-doh` flag accepts a comma-separated resolver list. Resolvers must use HTTPS without URL credentials or fragments; redirects are rejected. Discovery has bounded alias following and request deadlines. There is no resolver field in the normal editor. This mechanism obtains ECH configuration only: it does not replace the operating system resolver for ordinary endpoint address lookups.

The installed connector uses the foreign domain for the destination, SNI and WebSocket Host. There is no separate domain-fronting SNI option in the menu. ECH discovery starts from that same domain. Finding an ECH record is not proof that a real TLS handshake accepts it; use `godoftunctl doctor TUNNEL_ID` and then test client traffic. See [Cloudflare setup](CLOUDFLARE.md).

Fragment splits ClientHello writes; it does not change routing, unblock an unreachable IP or guarantee resistance to filtering. Browser profiles alter TLS/HTTP handshake characteristics; they do not turn the tunnel into a browser or guarantee indistinguishability.

The Iran `ca` setting verifies the **Cloudflare edge**. The separate `local_ca` setting is a trust bundle for local origin diagnostics, not the edge. Supplying an offline CA bundle is supported; skipping certificate verification is not. At core level, exactly one of `-ca`, `-system-ca` or `-fingerprint` is required for the outgoing connection. Managed installs use `-ca`; fingerprint/system-store modes are not extra installer menu choices.

## Local edits and paired edits

Shared fields are the two IPs, foreign domain and ports, WebSocket path, target port, token, MUX, padding and Iran default entry ports. Other tuning, trust and certificate settings are local. A local display-name change does not require peer synchronization.

For a shared change:

1. Open `edit` on one peer, select the field and confirm the change.
2. Transfer the emitted confidential edit code securely to the matching peer.
3. On that peer, run `sudo godoftunctl edit-apply TUNNEL_ID`, paste the code at the prompt and review the change.
4. Verify both peers before making another shared change. A temporary interruption is possible while settings differ.

Codes are HMAC-authenticated, tied to the pairing identity and opposite role, and checked against the prior value. They expire after seven days and reject stale or wrong-peer values; accurate server clocks matter. The first Iran-IP/entry-port synchronization may fill missing foreign-side metadata. Code contents are encoded, **not encrypted**; a token-rotation code contains the new secret.

`apply-update` is an alias of `edit-apply` in this release. It does not accept old unsigned/raw update formats or 3.x codes. Installation pairing codes from `peer-code` are a different format and are not editor codes. If synchronization fails, inspect both sides and resolve the stated mismatch; do not bypass validation or make unrelated shared edits on top of it.

## Customers, ports and lifetime quotas

Customer management runs on Iran:

```bash
sudo godoftunctl customers TUNNEL_ID
sudo godoftunctl usage TUNNEL_ID
```

The installer creates a `default` account covering the initial Iran entry ports, with an unlimited customer quota. The separate total tunnel quota defaults to `0`, also unlimited. Adding a customer suggests a 100 GB allowance; review that value before accepting it. A customer can later have several entry ports, all sharing that account's allowance.

Accounting follows the **entry port/account**, not an authenticated Xray UUID. Upload and download are added together. It counts bytes at the Iran client socket, excluding GODOFTUN framing and padding; it is not an ISP/CDN billing meter or a count of application files alone. Anyone who can use a different known entry may consume that entry's allowance. Configure actual client authentication at the foreign application and limit or disable the default entry if appropriate.

`1 GB = 1,000,000,000 bytes`. Fractional GB values must resolve to whole bytes. The management helper accepts a non-negative allowance up to 9,000,000 GB. `0` means unlimited, but an unlimited customer is still constrained by a finite total tunnel quota. Reaching either applicable limit blocks traffic; reducing a limit can close existing connections.

Quotas are lifetime ceilings, not recurring packages. If 30 GB has been used and the quota is changed to 50 GB, approximately 20 GB remains—not another 50 GB. Renaming the tunnel, editing ports, enabling/disabling a customer, restarting and normal upgrades do not reset usage. There is no monthly renewal, reset-usage or customer-delete command.

The ledger reserves bytes durably before allowing traffic. After a crash, unused reservations can conservatively count as usage, by up to 64 KiB per active account. This protects against undercounting across restarts; do not promise exact billing equivalence. Missing or invalid accounting state is an error, not a reason to initialize a fresh ledger.

Disabling an account closes its access while retaining its ID, quota, counters and port registration. Enabling restores access only if applicable quotas allow it. A stopped tunnel remains stopped when customer settings are saved. Customer-port edits preserve consumption and reject ports reserved by other accounts/tunnels, even if those entries are disabled.

Customer export files contain endpoint guidance, not server login, pairing tokens or complete Xray credentials. Supply the intended client credentials separately. Previously distributed files do not update on customer devices; export and redistribute changed endpoints. See [Operations](OPERATIONS.md) for the complete command list and [Security](SECURITY.md) before sharing files or diagnostics.

## Advanced runtime flags

The installed core's `godoftun -h` is the reference for flags outside the managed editor. Defaults include `dial-timeout=10s`, `auth-timeout=10s`, `open-timeout=15s`, `session-idle=90s` and `probe-timeout=15s`. Core limits are 1–60 seconds for dial/auth/write, 1–120 seconds for open, more than 30 seconds through one hour for session idle, and a positive probe timeout of at most 60 seconds. The installer logs stats every five seconds; the raw core defaults to stats disabled.

Do not confuse raw flags with editor ranges: for example, the core accepts edge-write timeouts up to one hour, while the managed editor offers up to 300 seconds. Direct launch-file changes may no longer pass managed checks. For everyday changes, use the editor and the values it actually shows.
