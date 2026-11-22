# Byte Balance — Portainer Templates

Custom Docker application templates for the Byte Balance VPS environment.

## Use with Portainer

Set the **App Templates** URL in Portainer to:

`https://raw.githubusercontent.com/kenanwahbeh/portainer-templates/main/templates.json`

The repository is public so Portainer can fetch the JSON file.

## Included templates

| Template | Purpose | Local-only host port |
|---|---|---:|
| PostgreSQL 17 | Relational database | 5432 |
| Redis 7 | Cache and in-memory data store | 6379 |
| Uptime Kuma | Uptime monitoring | 3001 |
| Vaultwarden | Bitwarden-compatible password manager | 8088 |
| Adminer | Database administration UI | 8081 |

## Network exposure

Published ports are explicitly bound to `127.0.0.1`, not to all host interfaces. This is intended for local access and Cloudflare Tunnel ingress configured on the VPS.

Do not change these bindings to `0.0.0.0` or expose administration interfaces directly to the public internet. For browser-based services, add suitable Cloudflare Access policies and HTTPS hostnames.

## Persistent data

Application data is stored under `/opt/bytebalance/<application>/`. Back up these directories before upgrades or migrations. Vaultwarden data is especially sensitive and must be protected.

## Important notes

- Templates create new containers and data directories; they do not update or replace existing services.
- Check available disk space and memory before deploying additional services.
- Set strong, unique passwords wherever prompted.
- Vaultwarden registrations are disabled by default. Temporarily enable registrations only for initial account creation, then disable them again.
- PostgreSQL uses the major version 17 image. Plan and perform a database migration before changing major versions.
- Review each image's official documentation and release notes before production use.
