# Operations — GODOFTUN V1.0.0

[English](OPERATIONS.md) · [فارسی](OPERATIONS.fa.md) · [简体中文](OPERATIONS.zh-CN.md) · [Русский](OPERATIONS.ru.md)

[Home](../README.md) · [Installation](INSTALLATION.md) · [Configuration](CONFIGURATION.md) · [Cloudflare](CLOUDFLARE.md) · [Troubleshooting](TROUBLESHOOTING.md) · [Security](SECURITY.md)

Commands below are instructions to run on your own servers, not a report that a deployment has been tested. Replace `TUNNEL_ID` and, where used, `CUSTOMER_ID` with actual IDs. Run management commands with root privileges. Read [Installation](INSTALLATION.md) before first setup and [Configuration](CONFIGURATION.md) before changing shared settings or quotas.

## Find the right tunnel

```bash
sudo godoftunctl list
sudo godoftunctl status TUNNEL_ID
sudo godoftunctl health TUNNEL_ID
sudo godoftunctl edit-show TUNNEL_ID
```

The friendly name is not the internal ID. A local service is named `godoftun@TUNNEL_ID`; its configuration is in `/etc/godoftun/tunnels/TUNNEL_ID`, and accounting state is in `/var/lib/godoftun/TUNNEL_ID`. Specify the ID explicitly in your own scripts. If it is omitted, the manager selects the only tunnel or asks you to choose among several.

## Installer commands

| Argument to `bash ./GODOFTUN-V1.0.0.sh` | Action |
|---|---|
| No argument | Main menu: foreign setup, Iran setup, management, transfer guide or V1 upgrade |
| `preview` or `--preview` | Display menus without root, network checks, installation or invented live status |
| `foreign` or `egress` | Install the foreign side; requires root |
| `iran` or `origin` | Install the Iran side; requires root and the foreign pairing code |
| `manage` | Open management for existing supported V1 tunnels; requires root |
| `upgrade` | Upgrade one selected supported V1 installation; requires root |
| `transfer-help` | Show file-transfer instructions |

The installed copy is `/opt/godoftun/share/godoftun-easy.sh`. Use a newly downloaded, verified release installer when upgrading; rerunning the old installed copy does not download a newer release.

## Complete `godoftunctl` command map

Unless noted, the form is `sudo godoftunctl COMMAND TUNNEL_ID`. `list` needs no ID; `help` prints usage. Commands that display information do not establish that real client traffic works.

| Command | Purpose |
|---|---|
| `list` | List tunnels on this server |
| `status` | Detailed systemd service status |
| `health` | Service state, PID/restarts, process resources, current-invocation stats and listeners |
| `monitor` | Refresh health every two seconds; exit with Ctrl+C |
| `logs` | Follow the last 100 journal entries and new entries; exit with Ctrl+C |
| `start`, `stop`, `restart` | Change only the selected tunnel service's running state |
| `config` | Display the managed launch command; normal installs use a token-file path rather than an inline token |
| `info` | Client endpoint and inbound guidance |
| `ports` | Registered ports; includes infrastructure reservations, not just open public client ports |
| `peer-code` | Display the confidential installation pairing code on the foreign side |
| `test` | Check the service and configured TLS path; not an end-to-end application test |
| `doctor`, `scan` | The same bounded diagnostic tool; options described below |
| `edit` | Open the full settings editor |
| `edit-show` | Display editable settings with the token hidden |
| `edit-ip` | Prompt for Iran or foreign IP and edit it |
| `edit-ports` | Edit foreign HTTPS ports on foreign, or default entry ports on Iran |
| `edit-apply`, `apply-update` | Apply a confidential signed V1 editor update; these are aliases |
| `customers` | Open customer management on Iran |
| `usage` | Show customer quotas, recorded usage, remaining allowance and status |
| `customer-add` | Add a separate customer entry and produce endpoint guidance |
| `customer-quota` | Change one customer's lifetime ceiling without resetting consumption |
| `customer-disable`, `customer-enable` | Change access state while retaining the account and consumption |
| `customer-export` | Write/display a customer's endpoint JSON file |
| `customer-ports` | Change one customer's direct entry port(s), retaining usage |
| `tunnel-quota` | Change the shared lifetime ceiling for the entire tunnel |
| `delete` | Remove one tunnel after typing its exact ID; see the removal section |
| `help` | Display command usage |

The existing-customer commands accept `CUSTOMER_ID` as a third positional argument; otherwise they ask you to choose. The port list and quota are requested interactively, not passed as undocumented extra CLI arguments. For example:

```bash
sudo godoftunctl customer-export TUNNEL_ID CUSTOMER_ID
sudo godoftunctl customer-quota TUNNEL_ID CUSTOMER_ID
sudo godoftunctl customer-ports TUNNEL_ID CUSTOMER_ID
```

Quota/port operations save changes after their prompts; do not assume every customer operation has the editor's separate `Apply?` confirmation. There is no `customer-delete`, monthly renewal or usage-reset command. See [customer accounting](CONFIGURATION.md#customers-ports-and-lifetime-quotas) before assigning allowances.

## Monitoring and bounded diagnostics

Start with `status` and `health`. Health shows current service-invocation stats, not old sessions from a previous run. Process RSS shown by system tools is different from the retained-buffer budget. `usage` marks stale/stopped snapshots; an old snapshot is not a live reading. A service being `active` is not proof of working Xray authentication, payload transfer or sustained speed.

The following checks do not change tunnel configuration. The first is offline; the others contact the configured or explicitly supplied endpoints:

```bash
sudo godoftunctl doctor TUNNEL_ID --offline
sudo godoftunctl doctor TUNNEL_ID --timeout 8
sudo godoftunctl doctor TUNNEL_ID --endpoints tunnel.example.com:443,tunnel.example.com:8443
sudo godoftunctl doctor TUNNEL_ID --output /root/godoftun-doctor.json
```

`tunnel.example.com` is a documentation example; replace it only with endpoints you own or are authorized to test. `doctor`/`scan` accepts at most 16 explicit host/IP candidates, not ranges. `--timeout` is 1–20 seconds, default 8, bounding each stage and the core probe's total duration. `--binary` is an advanced override for the matching executable; the default is `/usr/local/bin/godoftun`, never a `run.sh` launcher.

`--offline` checks local configuration without making network requests. It cannot certify connectivity. Online success requires a fresh authenticated carrier check; strict ECH uses the actual ECH-enabled core handshake. Even `carrier-authenticated` does not test the final application payload or customer credentials. Exit status is 0 for authenticated carrier or successful offline configuration check, 1 for an unsuccessful diagnostic verdict, and 2 for configuration/report errors.

`--output` creates a private, redacted JSON file and refuses to overwrite an existing file or symlink. Choose a fresh filename on subsequent runs. Review the report before sharing it. Raw logs, `config`, `info`, `edit-show` and screenshots can still expose addresses, paths or operational details. `peer-code`, pairing files, edit codes and backups are confidential. Never paste them into a public issue. See [Security](SECURITY.md) and [Troubleshooting](TROUBLESHOOTING.md).

## Apply a paired change

Use `edit` on one peer and `edit-apply` on the other. Transfer the printed code securely and paste it into the prompt—do not put it in a shell command or public message. Apply each shared change to its peer before starting another shared change. The other server's local tunnel ID may differ; match the actual pair, not just a similar display name.

Codes expire after seven days and check the prior value, pairing identity and opposite role. `apply-update` now calls the same signed editor; it is not a route to the old raw update mechanism. `peer-code` is for initial installation, not applying edits. Domain/IP changes still require your DNS and customer-device changes; endpoint files already sent to customers are not remotely updated. Details are in [Configuration](CONFIGURATION.md#local-edits-and-paired-edits).

## Transfer the installer when downloads are restricted

The complete installer already includes both Linux architectures. Copy the verified file from a trusted machine; Iran does not need Go or GitHub access just to receive the executable. The example addresses below are reserved for documentation and must be replaced with your own server addresses.

From a trusted machine, push the release file to Iran:

```bash
scp ./GODOFTUN-V1.0.0.sh root@203.0.113.20:/root/GODOFTUN-V1.0.0.sh
```

Alternatively, run this on Iran to pull the installed copy from your foreign server:

```bash
scp root@198.51.100.10:/opt/godoftun/share/godoftun-easy.sh /root/GODOFTUN-V1.0.0.sh
```

Only rename that installed copy as shown if it is the intended V1.0.0 installer. Verify the SSH host key and compare the received SHA-256 with your trusted original before running it:

```bash
sha256sum /root/GODOFTUN-V1.0.0.sh
sudo bash /root/GODOFTUN-V1.0.0.sh iran
```

This is offline **file delivery**, not a promise of completely offline installation: system packages, DNS, certificate checks and the working Cloudflare path may still require network access. Existing required packages must be available. The installer can use a valid PEM CA bundle placed at `/etc/godoftun/tunnels/TUNNEL_ID/ca-bundle.pem` or the shared `/etc/godoftun/ca-bundle.pem`; otherwise it uses the system CA bundle or tries to install `ca-certificates`. Transfer public CA certificates only from a trusted source, not a server's private key. See [Installation](INSTALLATION.md).

## Upgrade an existing V1 tunnel

Upgrade is for a registered `GODOFTUN_V1_CF_WSS` installation, not a 3.x migration. Unsupported fleet metadata, foreign direct/proxy alternatives or an unregistered existing executable block shared binary replacement. Do not remove those guards or relabel a 3.x installation as V1.

1. Read the new release's validation notes. Keep the previous verified installer and secure backups; schedule a maintenance window for both peers.
2. Run the new installer with `upgrade` on each server and select the intended local tunnel. Verify Cloudflare DNS when requested and review the confirmation.
3. Repeat for every existing tunnel on both servers. The executable and management helpers are shared, but only the selected tunnel's configuration and service are upgraded in that operation. Other services are not restarted automatically; a later restart uses the shared new executable.
4. Check `status`, `test` and `doctor`, then perform a real client upload/download, multiport, quota and reconnection test. Service checks alone do not certify the network path.

```bash
sudo bash ./GODOFTUN-V1.0.0.sh upgrade
```

Existing supported settings, certificates, IDs, quotas and consumption are retained. A stopped tunnel remains stopped. If accounting config exists but its ledger is missing, upgrade stops rather than granting fresh usage. A supported older V1 without accounting is prompted for a starting total allowance; metering begins at that upgrade because historical traffic cannot be reconstructed.

## Rollback, backups and accounting

Install/upgrade transactions, editor changes and customer changes keep scoped recovery copies under `/opt/godoftun/backups/`. Each operation prints its own backup location. A failed operation attempts to restore the files and service state it changed; when safe recovery cannot be confirmed, the affected service may be stopped and manual attention is required.

These are not interchangeable full-system snapshots. Editor/customer rollback deliberately avoids restoring an older live traffic ledger and refunding already consumed bytes. Do not copy old `access-state.json` over a running or newer ledger. Ledger corruption, missing state or failed acknowledgment must be investigated; deleting accounting files to make the service start loses the usage guarantee.

There is no documented `godoftunctl restore` or one-command downgrade. Keep protected copies of the relevant configuration, current accounting state, matching release and certificate material according to your backup process. A consistent manual recovery may require stopping the affected tunnel and coordinating both peers. Recover from the operation's actual backup and error report, not a guessed generic restore command. Backups contain secrets and belong in private storage.

## Stop, disable a customer, or remove a tunnel

These are different operations:

- `stop` stops one tunnel service but retains its installation, settings and usage. `start` brings it back.
- `customer-disable` disables an account and closes its access while preserving its identity, counters, quota and registered ports. `customer-enable` does not reset usage or override an exhausted quota.
- `delete` removes an entire selected tunnel from management; it is not a customer deletion or whole-program uninstall.

For removal, inspect the ID and run the following only when you intend to remove that tunnel:

```bash
sudo godoftunctl delete TUNNEL_ID
```

The manager asks you to type the exact tunnel ID. It stops and disables that service, moves its config and state into a private `deleted-TUNNEL_ID-...` backup, and removes its managed Nginx site only when it is not referenced by another tunnel. Nginx validation/reload is checked. A failure triggers an attempted rollback.

Shared executables, Nginx itself, other tunnels, certificates and firewall rules are retained. Review unused firewall openings yourself; do not assume deleting one tunnel safely closes every port. Deletion on one server does not delete its peer, DNS records or customer devices. The retained backup is recoverable data, not an automatic restore feature. Use [Security](SECURITY.md) for handling retained credentials and [Troubleshooting](TROUBLESHOOTING.md) if removal or recovery reports a failure.
