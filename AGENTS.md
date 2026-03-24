# AGENTS.md

This repo is a Docker Compose home-assistant stack with templated config.
Use this guide for commands and conventions when editing files here.

## Repository overview

- Root `compose.yml` includes service-specific `*/compose.yml` files.
- Each service directory generally matches its service name.
- Secrets live in `secrets.yml` (SOPS encrypted) with rules in `.sops.yaml`.
- Config and env files are generated from `*.gotmpl` templates.
- Shared config inputs are in `values.yml` and `secrets.yml`.

## Commands

### Templates and secrets

- `task tpl` runs `docker compose run --rm utils maketemplate` to render templates.
- `task sops` runs `sops edit secrets.yml` to edit encrypted secrets.

### Lint (YAML)

- `pre-commit install` to enable hooks locally.
- `pre-commit run check-yaml --all-files` to lint all YAML.
- `pre-commit run check-yaml --files path/to/file.yml` to lint one file.

### Run stack (local)

- `docker compose up -d` to start all services from root `compose.yml`.
- `docker compose up -d <service>` to start a single service.
- `docker compose ps` to check status.
- `docker compose logs -f <service>` to tail logs.
- `docker compose down` to stop the stack.

### Tests

- No unit or integration test runner is defined in this repo.
- Runtime readiness is enforced via Compose healthchecks.

### Single-file checks

- YAML: `pre-commit run check-yaml --files path/to/file.yml`
- Compose sanity (optional): `docker compose config -q`

## Configuration workflow

- Edit `values.yml` for non-secret settings.
- Edit `secrets.yml` via `task sops` for secret values.
- Run `task tpl` to re-render templated files after changes.
- Keep generated outputs committed when they are tracked in the repo.
- Ensure generated scripts remain executable if the template produces `.sh`.

## Environment files

- Root `.env` defines host paths and IPs consumed by Compose.
- `*.env.gotmpl` are templates; treat generated `.env` as outputs.
- Keep `.env.example` updated with new required variables.
- Use `${VAR}` substitutions in Compose; quote when special chars are present.
- Avoid placing real secrets in `.env.example`.

## Code style and conventions

### YAML (general)

- Use 2-space indentation (matches `.sops.yaml` and existing files).
- Use `#` comments and keep comment blocks aligned to their section.
- Keep one key per line; prefer block lists for long sequences.
- Use quotes for strings with special characters or templates (e.g., `${VAR}`).
- Keep blank lines between top-level sections for readability.

### Docker Compose files

- Service and directory names are lowercase (e.g., `homeassistant`, `adguardhome`).
- Keep `compose.yml` in each service directory.
- Prefer `env_file: - .env` and `${VAR}` substitutions over inline secrets.
- Keep key order consistent: `image`, `env_file`, `command`, `volumes`, `ports`,
  `restart`, `labels`, `healthcheck`, `depends_on`.
- Healthchecks should include `interval`, `timeout`, `retries`, `start_period`.
- Use `depends_on` with `condition: service_healthy` when ordering matters.

### Healthchecks

- Prefer HTTP checks when a service exposes HTTP.
- Use `CMD` or `CMD-SHELL` arrays for clarity.
- Keep timing values aligned with other services unless justified.
- Add `depends_on` entries when another service requires readiness.

### Logging and persistence

- Mount data volumes under `${DATA_ROOT}` for persistence.
- Mount logs under `${LOG_ROOT}` per existing pattern.
- Keep static config files in repo when they are not secret.
- Use read-only mounts (`:ro`) for config where possible.

### Values and secrets

- Update `values.yml` and `values.example.yml` together.
- Use lowerCamelCase keys in values (e.g., `cloudflareTunnel`, `pocketId`).
- Keep placeholder secrets as empty strings in `values.example.yml`.
- Edit `secrets.yml` only via `task sops` or `sops edit`.

### Traefik configuration

- Keep `traefik/traefik.yml` and `traefik/dynamic_conf.yml` in YAML style.
- Router names should match service keys for discoverability.
- Use Host rules with backtick-quoted domains (example: Host(`subdomain.example.com`)).
- Keep middleware lists aligned with similar services (e.g., `crowdsec`, `oauth2`).
- Prefer `service: <name>@docker` for container services, `@file` for host services.

### Templates (.env.gotmpl)

- Templates use gomplate syntax (`{{ ... }}`).
- Use one environment assignment per line (`KEY=value`).
- Use UPPER_SNAKE_CASE for environment variable names.
- Avoid trailing spaces; no quotes unless required by the value.

### Shell scripts

- Use POSIX `sh` with `#!/bin/sh`.
- Start scripts with `set -eu` for fail-fast behavior.
- Use functions for cleanup and `trap` for teardown.
- Indent with 2 spaces; avoid bash-only features.

### Imports, types, and structure (non-YAML)

- No application code is present today; keep new code minimal and justified.
- Group imports by standard library, third-party, then local modules.
- Add types when the language supports it (TypeScript, Python typing).
- Keep module and file names lowercase and match the service or feature.

### Naming conventions

- Directory and service names: lowercase, no spaces.
- Hostnames and subdomains derive from `values.yml` keys.
- Compose project name is `home-assistant` (root `compose.yml`).
- Config filenames use `compose.yml`, `.env`, `.env.gotmpl`, `config.yaml`.

### Error handling and safety

- Fail fast in scripts (`set -eu`) and surface errors.
- Prefer explicit healthchecks over implicit readiness.
- Do not disable healthchecks unless required by upstream behavior.
- Keep restart policies (`unless-stopped`) consistent with existing services.

## Security and secrets

- Never commit decrypted secrets or credentials.
- Prefer SOPS-managed `secrets.yml` and reference via templates.
- Sanitize logs if they might include tokens or passwords.
- Use `.env.example` for non-secret placeholders only.
- Keep encryption key material outside the repo.

## Agent workflow notes

- Generated files are overwritten by `task tpl`; edit templates and values instead.
- When adding a service, follow the existing directory pattern.
- Add new values to both `values.yml` and `values.example.yml`.
- Update root `compose.yml` to include new service `compose.yml` files.

## Common pitfalls

- Editing generated `.env` instead of the `.env.gotmpl` template.
- Forgetting to update `values.example.yml` when `values.yml` changes.
- Adding new services without including them in root `compose.yml`.
- Breaking healthchecks by changing ports without updating checks.
- Committing real secrets to `secrets.dec.yml` or `.env.example`.

## Review checklist

- `task tpl` rerun after template, values, or secrets edits.
- `pre-commit run check-yaml --all-files` passes.
- Root `compose.yml` includes new service files.
- Healthchecks and `depends_on` are consistent.
- No secrets appear in the git diff.
