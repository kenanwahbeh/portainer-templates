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

## Important notes

- Templates create new containers and data directories; they do not update or replace existing services.
- Check available disk space and memory before deploying additional services.
- Set strong, unique passwords wherever prompted.
- Vaultwarden registrations are disabled by default. Temporarily enable registrations only for initial account creation, then disable them again.
- PostgreSQL uses the major version 18 image. Major-version changes require a planned database migration. PostgreSQL 18 stores its data under `/var/lib/postgresql` inside the container.
- Review each image's official documentation and release notes before production use.
