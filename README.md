# Byte Balance — Portainer Templates

Custom Docker application templates for the Byte Balance VPS environment.

## Use with Portainer

Set the **App Templates** URL in Portainer to:

`https://raw.githubusercontent.com/kenanwahbeh/portainer-templates/main/templates.json`

The repository is public so Portainer can fetch the JSON file.

## Included templates

| Template | Purpose | Local-only host port |
|---|---|---:|
| PostgreSQL 18.7 | Relational database | 5432 |
| Redis 8.10.3 | Cache and in-memory data store | 6379 |
| Uptime Kuma 2.5.12 | Uptime monitoring | 3001 |
| Vaultwarden | Bitwarden-compatible password manager | 8088 |
| Adminer | Database administration UI | 8081 |

## Network exposure

Published ports are explicitly bound to `127.0.0.1`, not to all host interfaces. This is intended for local access and Cloudflare Tunnel ingress configured on the VPS.

Do not change these bindings to `0.0.0.0` or expose administration interfaces directly to the public internet. For browser-based services, add suitable Cloudflare Access policies and HTTPS hostnames.

## Persistent data

Templates use Docker-managed anonymous volumes: each template declares only the container path, without a host bind path or a manually assigned volume name. Portainer/Docker creates and manages the volume automatically.

On a standard Linux Docker installation, local Docker volumes are stored under `/var/lib/docker/volumes/`. The volume exists independently of the container, so its data can survive container recreation. When removing a container, do not select the option to remove its associated volumes unless you intend to delete the data. Vaultwarden data is especially sensitive and must be protected.

Adminer is a stateless database administration interface in this template and does not require a persistent data volume.

## Planned AI and messaging architecture

The following is the intended VPS architecture. These components are a design plan only; they are **not included as deployable templates yet**.

- **OpenCode Server** is the AI execution service and exposes its API only on a private Docker network.
- **Telegram User** connects a personal Telegram account using the Telegram client protocol (MTProto).
- **Telegram Bridge** receives authorized messages, maps conversations to OpenCode sessions, forwards requests to the OpenCode API, and returns responses to Telegram.
- **Vaultwarden** is the human-managed, encrypted password vault and the source of truth for API keys, passwords, and other credentials.

### Secret handling and manual startup

The VPS will use a manual post-reboot startup procedure for services that require credentials:

1. Docker and Portainer start.
2. Vaultwarden starts.
3. The operator manually unlocks the password vault using an authorized Bitwarden-compatible client.
4. The operator then performs the separately designed and secured secret-release/startup step for dependent services.
5. OpenCode Server and the Telegram services start only after their required credentials have been made available.

**Important:** Unlocking a Vaultwarden vault in a client does not unlock the Vaultwarden server for containers, nor does it automatically provide secrets to other containers. A dedicated, security-reviewed integration is required to retrieve and deliver secrets at runtime. Until that integration exists, do not claim that containers can read secrets directly from Vaultwarden.

Secrets must never be committed to this repository, embedded in template JSON, or placed in public Compose files. OpenCode's API must remain private to the Docker network; do not publish it directly to the public internet. Telegram access must be restricted to explicitly authorized users.

## Important notes

- Templates create new containers and Docker-managed volumes; they do not update or replace existing services.
- Check available disk space and memory before deploying additional services.
- Set strong, unique passwords wherever prompted.
- Vaultwarden registrations are disabled by default. Temporarily enable registrations only for initial account creation, then disable them again.
- PostgreSQL uses the major version 18 image. Major-version changes require a planned database migration. PostgreSQL 18 stores its data under `/var/lib/postgresql` inside the container.
- Review each image's official documentation and release notes before production use.
