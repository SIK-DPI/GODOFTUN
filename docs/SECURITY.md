# Security and public package contents

[فارسی](SECURITY.fa.md) · [简体中文](SECURITY.zh-CN.md) · [Русский](SECURITY.ru.md) · [Home](../README.md)

GODOFTUN V1.0.0 is distributed here as compiled executables inside a readable installer. This page explains what remains visible, what to keep private, and what the tests do not establish.

## What is and is not hidden

| Item | Public package |
|---|---|
| Original Go source and development tests | Not included |
| Bash installer and shell manager | Readable |
| Four Python management helpers | Readable inside the installer; installed as files on the server |
| Linux amd64 and arm64 executables | Embedded as decodable Base64, not encrypted |
| Go module/build metadata and linked symbols | May be inspected in the executables |
| Server-generated token, private key, pairing code and customer ledger | Not part of the release; created or supplied during deployment |

Compilation is not copy protection. A recipient can extract and analyze the binaries; Base64 is packaging, not secrecy. Removing comments or stripping symbols does not guarantee concealment. The Go symbol information is retained for meaningful vulnerability scanning. This is a binary distribution, not a claim that all implementation details are hidden.

## Files that must stay private

Keep the private owner-source ZIP, server backups and these deployment files out of the public repository:

- `/etc/godoftun/tunnels/TUNNEL_ID/token`, `peer.env`, `deployment.env` and the tunnel configuration directory.
- `/var/lib/godoftun/TUNNEL_ID/`, including usage ledgers and status snapshots.
- `/opt/godoftun/backups/`, certificate private keys and administrator SSH credentials.
- Pairing codes, signed edit codes, customer exports and unredacted logs/screenshots.

The pairing code contains a token. A signed edit code is authenticated but not encrypted and may contain sensitive values. Root access to a server can read the installed files; file permissions cannot hide them from that administrator. Do not upload the whole owner's output folder: use only the public ZIP or the contents of `PUBLIC_GITHUB`.

The included `.gitignore` helps prevent accidental additions, but does not erase already committed secrets. Review staged files and repository history before publishing. If a secret has escaped, remove the exposure and rotate the affected credential; deleting a post alone is not rotation. Coordinate tunnel-token changes across both peers using [the editor](CONFIGURATION.md), and expect an interruption while shared settings differ.

## Trust and access boundaries

Cloudflare terminates the outer TLS connection. Use proper application-layer encryption and authentication if the application must remain confidential from intermediaries. An Iran entry port or customer quota is not an Xray UUID/password. A person who knows another reachable entry can use that entry's allowance if the target application admits them.

Keep the foreign target and WSS backend on loopback. Review public firewall rules and provider rules. Never publish the tunnel token as a client credential or disable TLS verification to bypass a setup failure.

The installer requires root and can install packages, create a service account, write Nginx/systemd configuration, open rules on active UFW, issue certificates and start services. Its rollback protects tracked files, not every possible system change. Use dedicated test servers and private backups; see [operations](OPERATIONS.md).

## External network access

Normal operation contacts the configured foreign endpoint. Installation also uses OS package repositories, the Cloudflare IP-range API, and potentially `api.ipify.org` when local public-IP detection is insufficient. Automatic certificate issuance uses Certbot's CA service. ECH discovery uses the configured DoH resolvers; the defaults are `cloudflare-dns.com` then `dns.google`. These are operational dependencies, not evidence that the deployment is offline or invisible to those services.

## Verification and limits

Verify `SHA256SUMS` from the trusted release before running the installer. The installer also checks embedded binary hashes and the expected version. A checksum is not a digital signature and cannot authenticate an attacker-controlled download and manifest together.

Read [validation results](../VALIDATION.md) for actual tests and remaining Linux/Cloudflare checks. A clean scan cannot establish the absence of all vulnerabilities. ECH, padding and Fragment do not promise anonymity, censorship resistance or a particular speed. “Active” service state and a successful carrier handshake are not complete application tests.

## Support and distribution

Use [@SIKDPI](https://t.me/SIKDPI) for project contact. Do not post secrets in a public group or issue. First ask for an appropriate private contact if the report concerns a credential or security issue; no separate security inbox or response SLA is claimed here. The [troubleshooting checklist](TROUBLESHOOTING.md) explains what can be shared after review.

Dependency licenses are in `THIRD_PARTY_LICENSES/`. The project's own redistribution terms still need owner approval; the dependency notices do not grant a project license. Credit for development assistance is stated in [acknowledgments](../ACKNOWLEDGMENTS.md).
