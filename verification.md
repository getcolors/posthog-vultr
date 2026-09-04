# Live verification

The record of the live verification this deployment passed, per the workspace
Compute Provider Standard §7-8 and the plan that introduced it. Commands,
outcomes and timings only: no address, key id or token appears here.

## What was verified

| | |
|---|---|
| Date | 2026-09-04 (UTC) |
| Provider | Vultr, region `ams` |
| Plan | `vc2-6c-16gb`, os id `2284` (Ubuntu 24.04 LTS x64) |
| Package | `getcolors/posthog` at `ce05ed2` (pin commit `9002e42`) |
| Mode | keygen (no `vultr-ssh-keys`), compute name = profile |
| State | R2, `posthog-vultr/posthog-infrastructure.tfstate` |

## Sequence and outcomes

| # | Command | Outcome |
|---|---|---|
| 1 | `./green create` | **refused before any provider call**, exit 2: four required application credentials were not set (`COLORS_PAR_POSTHOG_SECRET_KEY`, `_POSTGRES_PASSWORD`, `_OIDC_RSA_PRIVATE_KEY`, `_ENCRYPTION_SALT_KEYS`). The sibling deployment's `colors.yml` header had never listed them; this deployment's header and README now do. Generated for this fresh deployment. |
| 2 | `./green create` | **exit 0.** Stages: start 0.7 s, infrastructure 84 s (instance, firewall group and rules, account key), ssh-config 2 s, dns 5 s, ansible 1178 s (ten containers, Postgres and ClickHouse migrations), acceptance 75 s (owner account provisioned, capture proven end to end). |
| 3 | `ssh posthog-vultr` | ten containers up; ClickHouse, Postgres, Kafka, Redis and Temporal healthy. The guest hostname reads `vultr`: the template sets no `hostname` on purpose, since that attribute is ForceNew and an OS reinstall. |
| 4 | `./red create` | **exit 0**, idempotent: infrastructure 4 s, ssh-config 2 s, dns 3 s, ansible 481 s, acceptance 66 s. |
| 5 | `./blue create` | **exit 0**, idempotent: infrastructure 4 s, ssh-config 2 s, dns 3 s, ansible 364 s, acceptance 68 s. |
| 6 | `COLORS_PAR_PROVIDER_COMPUTE=digitalocean ./green create` | **exit 2** before any credential or provider call: `state holds a vultr machine; set provider-compute back to vultr and delete first`. No DigitalOcean credential was demanded. |
| 7 | `COLORS_PAR_PROVIDER_COMPUTE=digitalocean ./green delete` | **exit 2**, the same refusal, ahead of the prevent-destroy guard. |

Attempt 1 is validation doing its job before anything is paid for. Attempt 2
was a fresh create; 4 and 5 prove the three colours manage one state
interchangeably; 6 and 7 prove Compute Provider Standard §4 on a live state.

## Not verified here

- The DigitalOcean side of the package, which `posthog-digitalocean` covers.
- The nightly backup timer and its restore path: the first backup set is
  written by the play, but the schedule had not fired within the verification
  window.
