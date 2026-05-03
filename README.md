# Foundry VTT Two-Tier Topology

This Podman Compose stack runs Caddy as a local HTTP origin for a Cloudflare Tunnel connector and Foundry VTT v14 as a private back-end service.

```text
Internet
   |
   | HTTPS
   v
Cloudflare
   |
   | Tunnel to local HTTP origin
   v
Caddy reverse proxy :8080
   |
   | backend container network
   v
Foundry VTT :30000
```

## Files

- `compose.yaml` defines the two service tiers and persistent container volumes.
- `Caddyfile` accepts local HTTP from Cloudflare Tunnel and proxies traffic to Foundry.
- `.env.example` lists the required settings.
- `.env` is your local, ignored configuration file.

## Configure

Create your local environment file if it does not already exist:

```bash
cp .env.example .env
```

Edit `.env` and set:

- `FOUNDRY_HOSTNAME` to the DNS name that points to this server.
- `FOUNDRY_RELEASE_URL` to a timed download URL from your Foundry account, or set `FOUNDRY_USERNAME` and `FOUNDRY_PASSWORD` instead.
- `FOUNDRY_LICENSE_KEY` to your Foundry license key.
- `FOUNDRY_ADMIN_KEY` to the admin password you want for Foundry.
- `CONTAINER_PRESERVE_CONFIG` to `true` only after you want Foundry options to stop following `.env` changes.

The default `.env` exposes only a localhost HTTP origin for Cloudflare:

```env
ORIGIN_HTTP_PORT=8080
FOUNDRY_PROXY_SSL=true
FOUNDRY_PROXY_PORT=443
```

In Cloudflare Tunnel, point the public hostname service at:

```text
http://localhost:8080
```

Cloudflare terminates public HTTPS. The connector talks to Caddy over local HTTP.

## Run

Install a Compose provider for Podman if one is not already present:

```bash
sudo dnf install podman-compose
```

Start the stack:

```bash
./foundry start
```

Watch startup logs:

```bash
podman compose logs -f
```

Open:

```text
https://your-foundry-hostname
```

For local smoke testing of the origin, open:

```text
http://localhost:8080
```


## Maintenance

Use the project wrapper for routine lifecycle commands. It preserves named
volumes and restarts services in dependency-safe order:

```bash
./foundry start
./foundry stop
./foundry restart
./foundry status
```

Avoid `podman compose restart` for this stack. Podman Compose can restart the
proxy while Foundry is in an improper dependency state. The wrapper stops the
proxy first, stops Foundry, starts Foundry, then starts the proxy.

Pull the newest compatible Foundry v14 container image and restart:

```bash
podman compose pull
./foundry restart
```

## Notes

- Only Caddy publishes a localhost HTTP origin port. Foundry is reachable only from the private `backend` container network.
- The Foundry container is pinned to the v14 major image line, `ghcr.io/felddy/foundryvtt:14`.
- Caddy does not request certificates in this topology. Cloudflare owns public TLS.
- Foundry data is stored in the `foundry_data` container volume.
- The origin is bound to `127.0.0.1` by default, so it is intended for the local Cloudflare connector rather than direct internet access.

# foundry
