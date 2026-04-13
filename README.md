# Project Compose Heimdall

A simple Docker Compose stack for running Traefik behind a Tailscale sidecar with optional Cloudflare Tunnel ingress.

The stack has three services:

- `heimdall-ts`: a Tailscale container that handles secure network access
- `heimdall-app`: a Traefik container that shares the Tailscale network namespace
- `heimdall-cf`: a Cloudflare Tunnel container that also shares the same network namespace and forwards traffic to Traefik

This setup supports two traffic paths to your local services:

1. Public Domain -> Cloudflare -> Traefik -> Local Service
2. Tailnet Device -> Tailscale Sidecar -> Traefik -> Local Service

## Requirements

- Docker
- Docker Compose
- A Tailscale auth key with the required tags configured
- A Cloudflare Tunnel token (for the public-domain path)
- DNS records for any hostnames used in your Traefik routes

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/jbledua/Project-Compose-Heimdall.git
cd Project-Compose-Heimdall
```

### 2. Create your environment file

Copy the example environment file to `.env`.

On Windows PowerShell:

```powershell
Copy-Item .env.example .env
```

On macOS or Linux:

```bash
cp .env.example .env
```

Edit `.env` and update these values:

- `STACK_NAME`
- `TIMEZONE`
- `TS_HOSTNAME`
- `TS_AUTHKEY`
- `TS_EXTRA_ARGS`
- `ACME_EMAIL`
- `CF_TUNNEL_TOKEN`
- `BOOKS_HOST`
- `PLEX_HOST`
- `BOOKS_URL`
- `PLEX_URL`

To create `TS_AUTHKEY`, open the Tailscale admin console and create an auth key:

https://tailscale.com/kb/1085/auth-keys/

If you are using tags, make sure the key is allowed to use the configured tag.

By default, `TS_EXTRA_ARGS` does not include `--reset`, which helps avoid restart loops on hosts that reuse the same auth key across restarts.

### 3. Configure route values

Set these environment values in `.env`:

- `BOOKS_HOST` and `PLEX_HOST` for public hostnames
- `BOOKS_URL` and `PLEX_URL` for backend endpoints

At startup, `heimdall-routes-init` generates Traefik dynamic routes into an internal Docker volume.

## Deploy the Stack

Run the stack from the directory containing `docker-compose.yml`:

```bash
docker compose up -d
```

This stack does not require host bind mounts for Traefik dynamic configuration, so it works in Portainer stack deployments that only include a compose file.

To check container status:

```bash
docker compose ps
```

To view logs:

```bash
docker compose logs -f
```

## Access

- Traefik dashboard is available through your configured ingress path
- Routed apps: use the hostnames configured in `BOOKS_HOST` and `PLEX_HOST`

If your client is connected to the same Tailscale tailnet, you can also access services using the node's Tailscale DNS name or Tailscale IP address.

For Cloudflare public access, create a named tunnel and set each public hostname's origin service to `https://localhost:443` (from the `heimdall-cf` container perspective), with origin certificate verification disabled if needed.

## Volumes and Data

The stack uses two named volumes:

- `heimdall-ts-state`: stores Tailscale state
- `heimdall-letsencrypt`: stores Traefik ACME certificates

## Notes

- The Traefik container uses `network_mode: service:heimdall-ts`, so it shares the Tailscale container's network stack.
- `ACME_EMAIL` from `.env` is passed to Traefik at startup using a Docker Compose command argument.
- Dynamic Traefik routes are generated at startup from `.env` values by `heimdall-routes-init`.
