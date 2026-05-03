# Foundry VTT Two-Tier Topology

This Podman Compose stack runs Caddy as the public reverse proxy and Foundry VTT v14 as a private back-end service.

```text
Internet
   |
   | 80/443
   v
Caddy reverse proxy
   |
   | backend container network
   v
Foundry VTT :30000
```

## Files

- `compose.yaml` defines the two service tiers and persistent container volumes.
- `Caddyfile` terminates TLS and proxies traffic to Foundry.
- `.env.example` lists the required settings.
- `.env` is your local, ignored configuration file.

## Configure

Create your local environment file if it does not already exist:

```bash
cp .env.example .env
```

Edit `.env` and set:

- `FOUNDRY_HOSTNAME` to the DNS name that points to this server.
- `ACME_EMAIL` to your email address for Let's Encrypt notices.
- `FOUNDRY_RELEASE_URL` to a timed download URL from your Foundry account, or set `FOUNDRY_USERNAME` and `FOUNDRY_PASSWORD` instead.
- `FOUNDRY_LICENSE_KEY` to your Foundry license key.
- `FOUNDRY_ADMIN_KEY` to the admin password you want for Foundry.
- `CONTAINER_PRESERVE_CONFIG` to `true` only after you want Foundry options to stop following `.env` changes.

The default `.env` uses rootless-friendly laptop ports:

```env
HTTP_PORT=8080
HTTPS_PORT=8443
FOUNDRY_PROXY_PORT=443
```

When you port forward, forward external TCP `80` to this laptop's `8080`, and external TCP `443` to this laptop's `8443`. Caddy still receives traffic on container ports `80` and `443`, so ACME HTTP-01 and normal HTTPS work as expected.

## Run

Install a Compose provider for Podman if one is not already present:

```bash
sudo dnf install podman-compose
```

Start the stack:

```bash
podman compose up -d
```

Watch startup logs:

```bash
podman compose logs -f
```

Open:

```text
https://your-foundry-hostname
```

For local smoke testing before DNS and port forwarding are complete, set `FOUNDRY_HOSTNAME=localhost` and open:

```text
https://localhost:8443
```

Your browser will warn about Caddy's local certificate unless you install Caddy's local CA.

## Maintenance

Pull the newest compatible Foundry v14 container image and restart:

```bash
podman compose pull
podman compose up -d
```

## Notes

- Only Caddy publishes host ports. Foundry is reachable only from the private `backend` container network.
- The Foundry container is pinned to the v14 major image line, `ghcr.io/felddy/foundryvtt:14`.
- Caddy will request and renew HTTPS certificates automatically when `FOUNDRY_HOSTNAME` resolves to this host and external ports 80/443 are forwarded to the configured laptop ports.
- Foundry data is stored in the `foundry_data` container volume.
- Rootless Podman may not be allowed to bind privileged host ports `80` and `443`. This setup defaults to `8080` and `8443` for that reason.

# foundry
