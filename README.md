# Home Assistant Compose Stack

Docker Compose stack for Home Assistant and supporting services, built for ARM64
and low resource hosts. I'm using an Orange Pi Zero 3 (4GiB RAM, 64GiB SD).

## Goals

- Keep resource usage low by only running what you need.
- Use templated config so values and secrets stay centralized.
- Make updates repeatable with simple task commands.

## Repository layout

- `compose.yml` includes per-service `*/compose.yml` files.
- `values.yml` (create from `values.example.yml`, not tracked) holds non-secret
  settings (domains, subdomains, ids).
- `secrets.yml` is SOPS-encrypted; edit via `task sops` only.
- `*.gotmpl` files render into runtime config and `.env` files.

## Services included by default

| Service                                                                        | Purpose                                     |
| ------------------------------------------------------------------------------ | ------------------------------------------- |
| [oauth2proxy](https://github.com/oauth2-proxy/oauth2-proxy)                    | Forward Auth Proxy                          |
| [pocketid](https://github.com/pocket-id/pocket-id)                             | Passkeys login                              |
| [rauthy](https://github.com/moonlight8978/rauthy)                              | OAuth federation, Central Identity Provider |
| [termix](https://github.com/lukegus/termix)                                    | Web Shell                                   |
| [traefik](https://github.com/traefik/traefik)                                  | Reverse Proxy                               |
| [cloudflaretunnel](https://github.com/cloudflare/cloudflared)                  | Allow Internet access to web services       |
| [cloudflareddns](https://github.com/oznu/docker-cloudflare-ddns)               | Cloudflare DDNS                             |
| [utils](https://github.com/moonlight8978/containers/tree/main/hac-toolkit)     | Render config, secrets from templates       |
| [crowdsec](https://github.com/crowdsecurity/crowdsec)                          | Threat Intelligence, WAF                    |
| [crowdsecfirewall](https://github.com/crowdsecurity/cs-firewall-bouncer)       | Firewall                                    |
| [portainer](https://github.com/portainer/portainer)                            | Monitor containers                          |
| [duplicity](https://github.com/moonlight8978/containers/tree/main/hac-toolkit) | Backup/Restore                              |
| [watchtower](https://github.com/nicholas-fedor/watchtower)                     | Auto-update docker images                   |
| [glances](https://github.com/nicolargo/glances)                                | Host monitoring                             |
| [acmesh](https://github.com/acmesh-official/acme.sh)                           | SSL auto renew                              |
| [adguardhome](https://github.com/AdguardTeam/AdGuardHome)                      | DNS server                                  |
| [homeassistant](https://github.com/home-assistant/core)                        | Home Assistant                              |
| [actual](https://github.com/actualbudget/actual)                               | Budget Management                           |
| [memos](https://github.com/usememos/memos)                                     | Notes taking app                            |
| [wgeasy](https://github.com/wg-easy/wg-easy)                                   | VPN for remote access                       |

To reduce resource use, comment out unneeded services in `compose.yml`.

## Quick start

1. Install prerequisites: Docker, Docker Compose plugin, `task`, `sops` (age).
2. Create `.env` from `.env.example` and update host paths and IPs.
3. Copy `values.example.yml` to `values.yml` and update values.
4. Edit secrets: `task sops`
5. Render templates: `task tpl`
6. Start the stack: `docker compose up -d`

Minimal start (Home Assistant + AdGuardHome):

```sh
docker compose up -d homeassistant adguardhome
```

Access Home Assistant at `http://<HOST_IP>:8123` (host network; typically the
same as `LAN_HOST`).

## Configuration workflow

1. Edit `values.yml` (created from `values.example.yml`) for non-secret
   settings.
2. Edit `secrets.yml` via `task sops` for secret values.
3. Run `task tpl` to re-render all templates.
4. Apply changes with `docker compose up -d <service>`.

Do not edit generated files directly. Edit the `*.gotmpl` templates instead.

## Resource-saving tips (Orange Pi Zero 3)

- Keep the include list in `compose.yml` minimal.
- Skip heavier services if you do not need them (crowdsec, duplicity,
  watchtower, glances, portainer, oauth2proxy, rauthy).
- Move `DATA_ROOT` and `LOG_ROOT` to external storage if possible to reduce
  SD card wear.
- Set `LAN_HOST` to a static IP so AdGuardHome can bind port 53 reliably.
- Prefer manual updates over continuous auto-updates on constrained hardware.

## Useful commands

```sh
# Render templates
task tpl

# Edit secrets with SOPS
task sops

# Start or restart services
docker compose up -d

# Check service status
docker compose ps

# Validate YAML
pre-commit run check-yaml --all-files
```

## Notes

- `utils` runs only when called (profile: dev) and is used by `task tpl`.
- Traefik binds to `LOCAL_HOST` (127.0.0.1 by default).
- Home Assistant uses host networking and depends on AdGuardHome health.
