# Troubleshooting GODOFTUN V1.0.0

[English](TROUBLESHOOTING.md) · [فارسی](TROUBLESHOOTING.fa.md) · [简体中文](TROUBLESHOOTING.zh-CN.md) · [Русский](TROUBLESHOOTING.ru.md)

[README](../README.md) · [Installation](INSTALLATION.md) · [Configuration](CONFIGURATION.md) · [Operations](OPERATIONS.md) · [Cloudflare](CLOUDFLARE.md) · [Security](SECURITY.md)

Start with the first failing stage and the affected tunnel. Reinstalling, replacing certificates, or restarting every service can hide the original cause. These checks do not promise a particular speed, availability, or resistance to filtering.

In commands below, replace `TUNNEL_ID` with the ID shown by `list`. Iran is `origin` in GODOFTUN logs; the foreign server is `egress`. Cloudflare's “origin server” means the foreign HTTPS server, not GODOFTUN's Iran role.

## Collect a small baseline

```bash
sudo godoftunctl list
sudo godoftunctl status TUNNEL_ID
sudo godoftunctl health TUNNEL_ID
sudo godoftunctl ports TUNNEL_ID
sudo godoftunctl doctor TUNNEL_ID --offline
sudo godoftunctl doctor TUNNEL_ID --timeout 8
sudo journalctl -u godoftun@TUNNEL_ID --since "15 minutes ago" -n 120 --no-pager
```

Run Doctor on both servers. `ports` lists registered ports; it does not establish that a process is listening. `status`, `health`, and `test` are useful checks, but an active service or an authenticated carrier is not a successful Xray upload/download test. Review logs before sharing them.

## Doctor commands and their limits

The installed wrapper accepts the following options after the tunnel ID. `scan` is an alias for `doctor`.

| Option | Meaning |
|---|---|
| `--offline` | Parse the selected configuration without making network requests. |
| `--endpoints HOST:PORT,...` | Test explicit candidates. Accepts IPs or domains, including `[IPv6]:PORT`; no URLs, CIDR ranges, or port ranges. An omitted port uses the first configured foreign port. |
| `--timeout N` | Integer seconds per stage, from 1 to 20; default 8. The core authentication attempt also has a bounded deadline. This is not a total time limit for the whole multi-endpoint report. |
| `--output FILE` | Create a sanitized JSON report with mode `0600`. The parent directory must exist; an existing file is never replaced. |
| `--binary PATH` | Use a matching GODOFTUN executable; default `/usr/local/bin/godoftun`. Do not pass `run.sh`. |
| `--help` | Show the helper's actual command help. |

For example, compare two ports on your configured domain:

```bash
sudo godoftunctl doctor TUNNEL_ID --endpoints tunnel.example.com:443,tunnel.example.com:8443 --timeout 8
sudo godoftunctl doctor TUNNEL_ID --output /root/godoftun-TUNNEL_ID-report.json
```

Use only endpoints you intend to test. Doctor retains the configured Host, SNI, path, and authentication when testing a candidate; it does not rewrite the tunnel. Testing an explicit IP is a diagnostic operation, not a supported direct-IP deployment mode. The report is limited to 16 tested candidates, including resolved addresses. Check `candidate_limit_reached` when DNS has many answers.

On Iran, the default targets are the configured foreign domain/port entries. On the foreign server, the default targets are local HTTPS on `127.0.0.1` and the Cloudflare edge, both on the **first** configured foreign port. Test other ports explicitly.

| Result | What it establishes |
|---|---|
| `configuration-checked-not-network-tested` | Offline parsing passed; networking was not tested. Exit status 0. |
| `carrier-authenticated` | At least one actual WSS/V1 authentication attempt passed, with no failed reported stage. Exit status 0. |
| `partial` | A carrier authenticated, but another reported stage or endpoint failed. Exit status 1. |
| `failed` | No candidate authenticated. Exit status 1. |
| `configuration-error` | Configuration could not be accepted, or a local input/report file could not be read or safely created. Exit status 2. |

`actual-carrier` uses the real core's TLS verification, browser profile, ECH/fragment setting, WebSocket upgrade, and V1 challenge-response. It opens no target stream. The report deliberately sets `target_payload_tested` and `cdn_origin_policy_verified` to false. Its recommended endpoint is the lowest single handshake time, not a throughput or reliability measurement.

## Installation stops before creating a tunnel

The installer requires root, Ubuntu or Debian, systemd, and a supported Linux amd64 or arm64 architecture. The packaged executable does not require Go on the server. Missing tools or dependencies may still require working package repositories. Read the exact missing-command or package error; the installer does not repair an already installed but broken Nginx package automatically.

For `flock is required`, provide the distribution's `util-linux` prerequisite through your normal server maintenance process. For `Another GODOFTUN installation is running`, wait for that operation to finish; do not delete its lock or launch a second installation.

For a port conflict, identify the owner before changing anything:

```bash
sudo ss -ltnp 'sport = :443'
sudo nginx -t
```

Replace `443` with the reported port. Choose an unused port through the installer/editor, or coordinate the existing service's configuration. A failing `nginx -t` can concern another site; do not remove unrelated sites to make the test pass. A domain already owned by another Nginx site needs a dedicated tunnel hostname. Keep the Go WebSocket backend on loopback. Do not modify an existing Native Reverse inbound to satisfy the new tunnel's target check; create the dedicated inbound described in [Installation](INSTALLATION.md).

## DNS does not resolve to Cloudflare

This installer requires the foreign hostname to resolve **only** to verified Cloudflare proxy addresses. A DNS-only record, an origin IP in an A/AAAA answer, stale resolver data, or failure to download Cloudflare's official IP ranges can stop preflight. The runtime domain, SNI, and WebSocket Host must agree.

```bash
getent ahosts tunnel.example.com
timedatectl status
```

Check both A and AAAA records, the selected hostname, Proxy ON, and the configured origin address in Cloudflare. Allow DNS changes to propagate, then retry the reported check. Failure to retrieve the official range list is a verification failure, not proof that the record is wrong. Do not replace the Iran connector's domain with the foreign IP or disable proxy verification. See [Cloudflare](CLOUDFLARE.md).

The installer accepts foreign HTTPS ports `443`, `2053`, `2083`, `2087`, `2096`, and `8443`. Iran client ports, the foreign HTTPS ports, the loopback Go backend port, and the Xray target port have different jobs.

## Locate the failing network stage

| Symptom or message | Check next |
|---|---|
| `DNS lookup failed` | Record and resolver reachability. No TLS test took place for that failed resolution. |
| `[tcp-connect]`, connection refused | The selected listener, its address/port, and an explicit network rejection. A certificate change cannot open a TCP listener. |
| Timeout or `deadline` | The displayed stage, route, server load, listener health, and scoped host/provider firewall rules. A timeout alone does not identify filtering, a token error, or a certificate fault. |
| `[tls-config]`, `[tls-handshake]`, `x509`, fingerprint mismatch | Trust source, certificate hostname/expiry, correct TLS leg, and matching pin where used. |
| `[tls-alpn] expected http/1.1` | The selected endpoint and TLS frontend. This carrier requires an HTTP/1.1 WebSocket upgrade. |
| WebSocket `404` | Exact private path, Host, forwarded Authorization/Upgrade headers, shared token, and clock. Rejected pre-authentication deliberately also returns 404. |
| `403` or `429` | Route-specific CDN challenge, WAF, access, or rate policy. This does not establish that the token is wrong. |
| `502`, `503`, `504`, `525`, or `526` | Cloudflare-to-foreign address/port, origin HTTPS/certificate, Nginx, and loopback backend health. The status alone does not identify the precise cause. |
| `expected challenge`, `egress rejected token`, unsupported protocol version | Matching V1 releases and the paired authentication configuration. WSS upgrade alone is insufficient. |

A browser challenge cannot be completed by the tunnel. Review the exact hostname/path policy in Cloudflare and the relevant event evidence; do not turn off account-wide security or the server firewall. The public decoy page responding at `/` does not prove the private tunnel route works.

## TLS succeeds on one leg but fails on the other

There are separate TLS connections from Iran to the Cloudflare edge and from Cloudflare to the foreign HTTPS server. Doctor additionally checks local foreign HTTPS. An edge certificate cannot fix a missing, mismatched, or expired origin certificate. A Cloudflare Origin CA certificate requires its appropriate CA bundle for the local origin check; it is not an ordinary public client trust certificate.

Check the certificate/key pair, DNS name, validity dates, service access to the files, and the selected CA or fingerprint policy. The core requires exactly one trust choice: system roots, an explicit CA file, or a certificate pin. Pins also check hostname and validity. Keep certificate verification enabled and follow [Security](SECURITY.md) and [Cloudflare](CLOUDFLARE.md) for the intended trust setup.

Both servers need correct clocks. More than five minutes of skew can reject the WebSocket pre-authentication even when the token is correct. Inspect the existing time service instead of repeatedly regenerating tokens.

## ECH or Fragment fails

Strict ECH needs both a usable DNS HTTPS ECH configuration and actual server acceptance. `godoftun -check-ech DOMAIN` checks configuration discovery/usability, not successful peer acceptance. Doctor's actual-carrier test checks the selected real handshake.

Strict ECH requires TLS 1.3 and cannot be combined with ClientHello fragmentation. A successful ordinary TLS probe, or a browser profile name, is not evidence that ECH worked. Doctor intentionally skips the ordinary TLS probe for ECH/Fragment profiles; that skipped stage is expected. It also skips a separate CA-only probe when the configured policy is certificate pinning.

For diagnosis, change one setting through `godoftunctl edit TUNNEL_ID`, coordinate any generated peer update, and rerun Doctor. Turning ECH off is an explicit profile change, not an automatic fallback. Fragment changes first-ClientHello TCP write boundaries; intermediaries can combine those writes. Neither setting guarantees connectivity or higher speed.

## Disconnects, restarts, or uneven traffic across ports

Compare both tunnel logs and `health` around the same timestamp. Check whether the service restarted, the remote session closed, a quota was reached, or a network timeout occurred. CDN/frontend restarts and connection failures require new sessions; reconnecting does not preserve an application's existing TCP stream. If you need to restart, use `sudo godoftunctl restart TUNNEL_ID` only after identifying the affected tunnel and accepting that its connections may be interrupted.

Multiple foreign ports are configured in priority order; they do not imply equal traffic on every port or automatic bandwidth aggregation. Test each intended foreign port with `--endpoints`. Test each Iran client port with the real client, and check the specific customer's port assignment and allowance. Multiple Iran ports can belong to one customer and share its quota.

For resource pressure, compare actual process memory with the retained-buffer limits described in [Configuration](CONFIGURATION.md). A configured memory budget is not a complete process RSS cap. Raising session/stream limits without checking memory and the target service can make failures worse.

## A paired edit is rejected or only one peer changed

Use `sudo godoftunctl edit-show TUNNEL_ID` on both sides to compare non-secret settings. The current editor's signed codes belong to a specific pair and direction, expire after seven days, and verify the previous value. A later edit, wrong tunnel, wrong peer, clock error, or a V3/3.x code can produce `Invalid, stale, expired, or wrong-peer update code`.

Confirm the correct pair, resolve the differing values, then generate a fresh V1 change from the appropriate server and apply it with `sudo godoftunctl edit-apply TUNNEL_ID`. `apply-update` is an alias. Do not alter the encoded code, change pair identity, or paste secrets into an issue. A local edit may interrupt traffic until its corresponding peer change is applied; follow [Operations](OPERATIONS.md).

## A customer is blocked or usage looks wrong

On Iran, run:

```bash
sudo godoftunctl usage TUNNEL_ID
sudo godoftunctl customers TUNNEL_ID
```

`disabled`, `exhausted`, `stopped`, `status stale`, `status unavailable`, and `blocked` describe different conditions. Check the customer's enabled state, its lifetime allowance, and the tunnel's total lifetime allowance. Upload and download are added together; `0` means unlimited for that allowance, and 1 GB is 1,000,000,000 bytes. Changing a quota retains usage, so a replacement quota below recorded usage does not unblock the customer.

The manager treats the status file as fresh only within roughly five seconds of its modification time. A stopped service, stale snapshot, or missing snapshot is not a live zero-usage reading. `The service did not acknowledge the new customer configuration` means the requested revision was not confirmed; inspect the selected service and the reported accounting error before retrying.

The durable ledger is `/var/lib/godoftun/TUNNEL_ID/access-state.json`. Errors such as `access state unavailable; refusing to reset usage`, checksum mismatch, a non-private state file, or `access state already in use` require checking the matching state/configuration, ownership/permissions, storage, and duplicate processes. Do not delete or recreate the ledger to make the service start. Keep its backup and use the recovery procedure in [Operations](OPERATIONS.md).

Accounting measures socket upload/download bytes, excluding tunnel framing and padding. Durable reservations can conservatively retain up to 64 KiB per active account after a crash; these counters need not match the provider's total interface traffic. Repeatedly resetting state would corrupt lifetime accounting.

## Upgrade or rollback fails

Keep the printed recovery directory under `/opt/godoftun/backups`. Read the first upgrade error and the rollback result. A restored configuration does not prove that the network path is healthy; rerun status, Doctor, and a real client test.

The executable and management helpers are shared, while tunnel configurations and services are individual. Upgrade the paired servers and each affected tunnel as described in [Operations](OPERATIONS.md). Unsupported profiles or 3.x installations can block shared-binary replacement; do not bypass that audit or mix pairing codes/releases.

If rollback reports manual attention, preserve both original and failed files, plus the current accounting ledger. Do not copy an old usage snapshot over a newer ledger or restore an entire server's Nginx configuration blindly. Automatic rollback restores managed files/service state where possible; downloaded dependencies, the service account, and issued certificates may remain. The installer does not modify Xray. A missing accounting state is deliberately not treated as a new zero-usage installation.

## Doctor passes, but the Xray client does not work

Check the remaining application path in order:

1. The client uses the Iran IP and its assigned customer port, with the protocol, credentials, transport, and TLS settings of the dedicated foreign Xray inbound.
2. That customer is enabled and both customer/tunnel allowances permit traffic.
3. The foreign target actually listens on the configured loopback address and port, and the egress target allow-list includes that exact target.
4. The Xray inbound accepts the supplied client settings, and its outbound/DNS route works.
5. A real client can upload and download; test short and sustained transfers on each intended entry port.

GODOFTUN's WSS carrier does not automatically change the Xray inbound's protocol or TLS configuration. Keep the dedicated inbound separate from existing services. See [Installation](INSTALLATION.md) for the topology and [Configuration](CONFIGURATION.md) for port roles.

## Send a redacted support report

Include both peers' `godoftun -version`, OS/architecture, tunnel roles, the failing stage and UTC timestamp, relevant port roles, whether ECH/Fragment/MUX are enabled, the last relevant change, and short logs for the affected tunnel. Include Doctor's verdict and stage results, plus whether a real client upload/download passed. If reporting accounting trouble, include the displayed state and snapshot age rather than the ledger itself.

Doctor sanitizes known secrets, but its report still contains endpoint addresses, SNI, tunnel ID, and diagnostic metadata. Review it before sharing. Remove tokens, Authorization headers, private WebSocket paths, pairing/update codes, private keys, customer exports/credentials, full launch/configuration files, and accounting files. Do not attach the entire backup directory. An existing `--output` file is never overwritten; choose a new private filename for a new report. See [Security](SECURITY.md) and the release's [validation limits](../VALIDATION.md).
