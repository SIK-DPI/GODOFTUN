# Configure the foreign Cloudflare domain

[فارسی](CLOUDFLARE.fa.md) · [简体中文](CLOUDFLARE.zh-CN.md) · [Русский](CLOUDFLARE.ru.md) · [Home](../README.md)

Use a dedicated subdomain for GODOFTUN V1.0.0. These settings are made in your Cloudflare account; the installer does not create DNS records or change account settings. Dashboard labels can move. The linked official documentation was checked on 2026-10-01.

## DNS

In the zone's DNS records, create an A record for your chosen subdomain pointing to the foreign server's public IPv4 address. Set Proxy status to Proxied, the orange cloud. With proxying enabled, DNS returns Cloudflare addresses rather than the server address. [Proxy status](https://developers.cloudflare.com/dns/proxy-status/)

For the first deployment use one intended origin. Do not add an unrelated AAAA record, load balancer or Worker route to this name. Review any existing records before changing them. Iran needs no DNS record.

On both servers, replace the example domain and inspect the answers:

```bash
getent ahostsv4 tunnel.example.com
getent ahostsv6 tunnel.example.com
```

GODOFTUN checks resolved addresses against Cloudflare's official ranges from `https://api.cloudflare.com/client/v4/ips`. A mixed, direct-origin, non-Cloudflare or unverified answer is rejected. If the API is unreachable, fix access and retry; this version has no offline bypass for that check.

## HTTPS ports and WebSockets

Start with TCP 443. GODOFTUN accepts HTTPS ports `443,2053,2083,2087,2096,8443`, matching Cloudflare's supported proxy ports. [Network ports](https://developers.cloudflare.com/fundamentals/reference/network-ports/)

In Network settings, enable WebSockets. The initial upgrade can be affected by security rules, and Cloudflare can close long-lived connections. [WebSockets](https://developers.cloudflare.com/network/websockets/)

For this deployment, preserve the domain's Host/SNI and the private path. Do not apply redirects, caching, interactive browser challenges or header rewrites to the authenticated WSS request. If a security rule blocks it, investigate the specific event and make the narrowest justified exception for your tunnel hostname/path. Do not disable protection across the entire zone. GODOFTUN generates its Nginx upgrade route; manually replacing that route can break authentication.

## Two separate certificate checks

The Iran connector verifies the public edge certificate. Nginx on the foreign server needs a separate origin certificate. Select Full (strict) in SSL/TLS settings and use a current origin certificate covering the hostname, signed by a publicly trusted CA or Cloudflare Origin CA. Passing a local CA-bundle check alone does not establish Cloudflare's trust. [Full strict mode](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/full-strict/)

The installer offers two origin-certificate methods:

| Method | What you provide |
|---|---|
| Automatic Let's Encrypt | Email and a reachable HTTP challenge path; the installer requests the certificate through Certbot |
| Existing certificate | Absolute certificate-chain and unencrypted private-key paths without spaces, plus the CA bundle used for local origin verification |

The local check verifies hostname, key match and expiry; a supplied certificate with less than 24 hours remaining is rejected. Do not paste private keys into the terminal prompts: enter paths to privately stored files.

For a Cloudflare Origin CA certificate, use its matching CA root bundle for local origin checks. Origin CA is not ordinary browser trust. [Origin CA](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca/)

The Iran connector still uses the public edge trust store, not your Origin CA bundle. The editor exposes separate local-origin and edge-CA fields. Do not solve certificate errors by disabling verification.

## First certificate and renewal

Automatic issuance uses the foreign server's HTTP webroot at `/.well-known/acme-challenge/`. TCP 80 must be reachable. Ensure this path reaches the intended origin without a challenge, cache or redirect to an HTTPS listener that does not yet exist. Keep Proxy ON. Limit any temporary routing/redirect exception to the challenge path and verify normal Full (strict) HTTPS before completing installation.

The installer can create Certbot renewal support and a managed Nginx reload hook. Existing non-Certbot certificates remain your responsibility. Monitor expiry and test the renewal arrangement during your Linux acceptance test. Do not assume success at installation proves future renewal.

If your account rules cannot support HTTP challenge bootstrap without weakening unrelated services, select an already issued certificate. The installer does not provide a DNS API challenge workflow or ask for your Cloudflare API token.

## ECH

Cloudflare documents ECH under SSL/TLS edge certificates. Availability/settings depend on the zone; ECH protects the inner ClientHello hostname from intermediaries, not from Cloudflare. [ECH configuration](https://developers.cloudflare.com/ssl/edge-certificates/ech/)

GODOFTUN's strict ECH mode needs an advertised HTTPS-record ECH configuration, reachable discovery and a TLS 1.3 handshake that actually accepts it. Enabling a dashboard setting alone is not proof. Use the actual GODOFTUN connection and Doctor results; a website visit or generic TLS check does not reproduce its handshake. See [ECH and Fragment settings](CONFIGURATION.md).

## Confirm the whole path

After foreign installation, a normal HTTPS request to the domain root should reach the generated generic site. Its “All systems operational” text is static cover-page text, not a live health result. Then complete Iran installation and a real application transfer as described in [installation](INSTALLATION.md).

Keep the origin address and application target protected. Cloudflare terminates the WSS TLS leg, so this is not end-to-end secrecy from Cloudflare; the application inside the tunnel must provide any required end-to-end encryption. Read [security](SECURITY.md) before distributing client access.
