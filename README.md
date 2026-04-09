# Project Compose Heimdall

A simple Docker Compose stack for running Traefik behind a Tailscale sidecar.

The stack has two services:

- `heimdall-ts`: a Tailscale container that handles secure network access
- `heimdall-app`: a Traefik container that shares the Tailscale network namespace

This setup lets you publish self-hosted services with Traefik while keeping network access controlled through Tailscale.

## Requirements

- Docker
- Docker Compose
- A Tailscale auth key with the required tags configured
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
- `HTTP_PORT`
- `HTTPS_PORT`
- `TRAEFIK_DASHBOARD_PORT`
- `ACME_EMAIL`

To create `TS_AUTHKEY`, open the Tailscale admin console and create an auth key:

https://tailscale.com/kb/1085/auth-keys/

If you are using tags, make sure the key is allowed to use the configured tag.

### 3. Create your routes file

Copy the example dynamic routes file and replace the placeholder values.

On Windows PowerShell:

```powershell
Copy-Item .\traefik\dynamic\routes.example.yml .\traefik\dynamic\routes.yml
```

On macOS or Linux:

```bash
cp ./traefik/dynamic/routes.example.yml ./traefik/dynamic/routes.yml
```

Then edit `traefik/dynamic/routes.yml`:

- Replace `app1.example.com` and `app2.example.com` with your real hostnames
- Update backend URLs (for example `http://host.docker.internal:12345`) to match your local services

## Deploy the Stack

Run the stack from the directory containing `docker-compose.yml`:

```bash
docker compose up -d
```

To check container status:

```bash
docker compose ps
```

To view logs:

```bash
docker compose logs -f
```

## Access

- Traefik dashboard: `http://localhost:8080` (or the value of `TRAEFIK_DASHBOARD_PORT`)
- Routed apps: use the hostnames configured in `traefik/dynamic/routes.yml`

If your client is connected to the same Tailscale tailnet, you can also access services using the node's Tailscale DNS name or Tailscale IP address.

## Volumes and Data

The stack uses two named volumes:

- `heimdall-ts-state`: stores Tailscale state
- `heimdall-letsencrypt`: stores Traefik ACME certificates

## Notes

- The Traefik container uses `network_mode: service:heimdall-ts`, so it shares the Tailscale container's network stack.
- `ACME_EMAIL` from `.env` is passed to Traefik at startup using a Docker Compose command argument.
- `traefik/dynamic/routes.yml` is gitignored so private hostnames and internal endpoints are not committed.
