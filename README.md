# Byte Balance — Portainer Templates

Custom Docker application templates for the Byte Balance Server environment.

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

These are the five templates currently defined in `templates.json`. OpenCode, Telegram services, MEGA CMD, and Portainer itself are not deployable templates in this repository.

## Network exposure

Published ports are explicitly bound to `127.0.0.1`, not to all host interfaces. This is intended for local access and Cloudflare Tunnel ingress configured on the Server.

Do not change these bindings to `0.0.0.0` or expose administration interfaces directly to the public internet. For browser-based services, add suitable Cloudflare Access policies and HTTPS hostnames. Services that only need to communicate with one another should use private Docker networking and should not publish host ports.

## Persistent data and Docker storage

Templates use Docker-managed anonymous volumes: each template declares only the container path, without a host bind path or a manually assigned volume name. Portainer/Docker creates and manages the volume automatically. No custom host data directories are defined by these templates.

On a standard Linux Docker installation, local Docker volumes are stored under Docker's default data root (normally `/var/lib/docker/volumes/`). The volume exists independently of the container, so its data can survive container recreation. When removing a container, do not select the option to remove its associated volumes unless you intend to delete the data. Vaultwarden data is especially sensitive and must be protected.

Adminer is a stateless database administration interface in this template and does not require a persistent data volume.

The current templates ask Portainer for some first-run service settings (for example, PostgreSQL and Redis passwords) through their template environment fields. This is separate from the planned runtime secret-delivery design for application integrations: MEGA credentials and other service-to-service secrets must not be hard-coded in Git, images, template JSON, or ordinary Compose configuration.

## Planned Server architecture

This section records the agreed target design. **It is documentation, not evidence that these components or automations have been implemented or deployed.** The repository currently contains only `README.md`, `SECURITY.md`, and `templates.json`; it has no Dockerfiles, unlock-agent source code, MEGA CMD integration, backup scripts, or deployment automation.

### Vaultwarden and automatic startup

The target operating model is **automatic startup and automatic vault unlock after every Server reboot**. Normal recovery should not require a person to manually unlock the vault.

1. Docker and Portainer start automatically.
2. The Vaultwarden server container starts automatically.
3. A dedicated, restricted secret-unlock agent starts only after Vaultwarden is healthy.
4. The agent uses a bootstrap credential protected **outside Vaultwarden** to authenticate an approved Bitwarden-compatible client/session and decrypt the user's vault.
5. It retrieves only the specific secrets required by dependent services and delivers them through a protected runtime mechanism.
6. OpenCode Server, Telegram services, and the MEGA CMD integration start or become ready only after their required secrets have been delivered and their readiness checks pass.

**Vault terminology:** Vaultwarden is a server and does not have a global server-side “locked/unlocked” switch. Unlocking means decrypting the user's encrypted vault in an authenticated client/session. A running Vaultwarden container does not, by itself, mean the vault has been unlocked or that application secrets are available.

**Bootstrap credential:** The credential needed to unlock the vault cannot be stored inside the same vault it unlocks. It must be protected independently on the Server and readable only by the restricted unlock agent. It must not be committed to Git, embedded in container images or template JSON, or placed in ordinary environment variables or public Compose files. The exact host-side protection, client integration, and runtime secret-delivery mechanism still require implementation and security review. A compromise of the Server or a privileged account may expose this bootstrap credential and the secrets available to the agent.

**Fail-closed behavior:** If Vaultwarden is unavailable, authentication/decryption fails, or a required secret cannot be retrieved, dependent services must remain stopped or unready. There must be no fallback to hard-coded credentials and no startup with missing secrets. Orchestration should distinguish:
- `STARTING`: Vault/session and required secret delivery are not complete.
- `READY`: authenticated vault session is established and the required secrets have been delivered.
- `LOCKED/UNAVAILABLE`: secrets cannot safely be obtained.

These are orchestration states, not native Vaultwarden server states.

### MEGA CMD and backup flow

The agreed direction is a **separate MEGA CMD container**, not a host-level MEGA CMD installation on Ubuntu.

- MEGA CMD is intended to upload and retrieve backup archives. It is not a database and must not become the only copy of important data.
- Its MEGA account credential must be retrieved at runtime from the Vaultwarden-backed secret flow. Do not put the password in the image, Git, template JSON, a public Compose file, or ordinary environment variables.
- The MEGA CMD container should have its own Docker-managed persistent volume for its authenticated session/state, so a container recreation does not automatically discard the session.
- No public host port is required for MEGA CMD. It should use outbound connectivity and private Docker networking only.
- Backups should be encrypted before upload. Restore must include access to the corresponding decryption material, protected separately from the encrypted backup.
- A container image discussed as a possible MEGA CMD image is only a candidate until its publisher, source, supported architecture, secret-file handling, and persistence behavior have been verified. No MEGA CMD image or integration is currently defined in `templates.json`.

### Portainer image and data backups

A Portainer recovery set has two separate parts:

1. **Container image:** the Portainer image and its exact version/tag (preferably also its immutable digest) can be pushed to the Docker Hub repository. Docker Hub stores the image, not Portainer's live application data.
2. **Portainer data:** the contents of the Docker-managed `portainer_data` volume mounted at `/data` must be backed up separately as an archive. The archive should be encrypted before it is uploaded to MEGA.

A usable recovery procedure therefore needs both the matching Portainer image and the separately protected data-volume backup. Uploading the image alone is not a complete Portainer backup. The backup and restore scripts/procedure have not yet been implemented or tested by this repository.

### OpenCode and Telegram

- **OpenCode Server** is the AI execution service. Its API must remain private on a Docker network and must not be published directly to the public internet.
- **Telegram User** connects a personal Telegram account using the Telegram client protocol (MTProto).
- **Telegram Bridge** receives authorized messages, maps conversations to OpenCode sessions, forwards requests to the private OpenCode API, and returns responses to Telegram.
- Telegram access must be restricted to explicitly authorized users.

These services are part of the target architecture only; they are not included as deployable templates yet.

## Important notes

- Templates create new containers and Docker-managed volumes; they do not update or replace existing services.
- Check available disk space and memory before deploying additional services.
- Set strong, unique passwords wherever prompted.
- Vaultwarden registrations are disabled by default. Temporarily enable registrations only for initial account creation, then disable them again.
- PostgreSQL uses the major version 18 image. Major-version changes require a planned database migration. PostgreSQL 18 stores its data under `/var/lib/postgresql` inside the container.
- Review each image's official documentation and release notes before production use.
- Do not treat a design described in this README as deployed until its implementation, configuration, and recovery behavior have been verified on the Server.
