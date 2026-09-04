# posthog-vultr

Desired state for one single-node [PostHog](https://github.com/getcolors/posthog)
product analytics suite on a Vultr instance in Amsterdam, published at
**https://posthog-vultr.bigconfig.online**.

This repository holds configuration, not code. The behaviour lives in the
`package-posthog-green`, `package-posthog-red` and `package-posthog-blue`
Package Skills installed under `.agents/skills/`; the three colours are
interchangeable managers of the same state.

It is the posthog package's second-provider pilot under the workspace Compute
Provider Standard: the same package that runs `posthog-digitalocean`, selected
onto Vultr by `provider-compute` alone.

## Architecture

- **Host**: `posthog-vultr.bigconfig.online`
- **Instance**: `vc2-6c-16gb` in `ams` (Ubuntu 24.04), Vultr's smallest
  16 GiB plan; PostHog's own hobby minimum is 16 GiB
- **Services**: ten containers — the PostHog web process and Celery worker, a
  standalone Rust `capture` service, the Node plugin server, Redpanda, Temporal,
  PostgreSQL, ClickHouse with embedded Keeper, Redis, and Caddy. None is
  optional; `../posthog` explains why each is required.
- **Disaster recovery**: nightly Postgres `pg_dump` and native ClickHouse
  `BACKUP`, uploaded to Cloudflare R2 (`posthog-backup`) under the profile prefix

Exact image pins are in `colors.yml`, which is the only file to edit here.

## Operations

```sh
direnv allow                 # once, after cloning
./green build                # render .colors/ — no provider calls, no credentials
./green create --dry-run
./green create               # converge for real
```

## Configuration

`colors.yml` holds non-secret values only. Credentials are `COLORS_PAR_*`
variables in the gitignored `.envrc.private`:

| Credential | Variable |
|---|---|
| Vultr API key | `COLORS_PAR_VULTR_API_KEY` |
| Cloudflare API token (`bigconfig.online` zone) | `COLORS_PAR_CLOUDFLARE_API_TOKEN` |
| R2 state backend | `COLORS_PAR_R2_ACCESS_KEY_ID`, `COLORS_PAR_R2_SECRET_ACCESS_KEY` |
| R2 backups (`posthog-backup` bucket) | `COLORS_PAR_POSTHOG_BACKUP_R2_ACCESS_KEY_ID`, `COLORS_PAR_POSTHOG_BACKUP_R2_SECRET_ACCESS_KEY` |
| Django signing key | `COLORS_PAR_POSTHOG_SECRET_KEY` |
| Postgres password | `COLORS_PAR_POSTHOG_POSTGRES_PASSWORD` |
| OIDC RSA private key (PEM, multi-line) | `COLORS_PAR_POSTHOG_OIDC_RSA_PRIVATE_KEY` |
| Encryption salt keys (32 hex chars) | `COLORS_PAR_POSTHOG_ENCRYPTION_SALT_KEYS` |
| PostHog owner password | `COLORS_PAR_POSTHOG_ADMIN_PASSWORD` |

None of the application secrets is optional; the package's configuration
reference explains what each one is for.

## SSH access

The deployment owns its keypair at `~/.ssh/posthog-vultr`, outside this
checkout, and the Vultr account key named `posthog-vultr` belongs to its
OpenTofu state. Cloning this repository elsewhere does not carry access — copy
the keypair deliberately. See `CLAUDE.md` for the failure modes.

Convergence also writes a `~/.ssh/config` block, so `ssh posthog-vultr`
connects with no address or flags.

## Safety

`compute-prevent-destroy: true` guards deletion; lifting it requires a one-run
`COLORS_PAR_COMPUTE_PREVENT_DESTROY=false` override and separate authorization.
Never export `COLORS_PAR_PROFILE`, and never edit or commit `.colors/`.
Changing `provider-compute` on this profile is refused while a machine is in
state: a provider switch is a delete followed by a create, never an apply.
