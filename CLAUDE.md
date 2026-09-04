# CLAUDE.md

## Repository

Desired state for `posthog-vultr`: one single-node PostHog product analytics
suite on a Vultr instance in Amsterdam, published at
`https://posthog-vultr.bigconfig.online` through Cloudflare and Caddy.
Behavior lives in `../posthog`.

This is the second-provider pilot of the workspace Compute Provider Standard
(`../workspace/standards/compute-provider.md`): the same package that runs
`posthog-digitalocean`, on Vultr by `provider-compute: vultr` and the
`vultr-*` keys alone. The hostname is deliberately distinct from
`posthog-digitalocean`'s so a rebuild of either can never collide with the
other.

Tracked source is `colors.yml`, toolchain and documentation, the three
installed Package Skills, the lockfile, and root launchers copied from their
payloads. `.colors/` is generated private state and `.envrc.private` contains
credentials; never read, edit or commit either.

## The machine keypair is not in this checkout

This deployment runs in **keygen mode**: `colors.yml` carries no
`vultr-ssh-keys`, so the package owns the keypair per the workspace SSH
Keypair Standard. It lives at `~/.ssh/posthog-vultr`(`.pub`) — outside this
repository — and the Vultr account key named `posthog-vultr` belongs to this
deployment's OpenTofu state.

- Cloning this repository on another workstation does **not** carry machine
  access. Copy `~/.ssh/posthog-vultr`(`.pub`) deliberately, or `create` will
  refuse rather than regenerate a key that cannot reach the live host.
- Never delete the Vultr key named `posthog-vultr` while the instance lives.
- Adding `vultr-ssh-keys` to `colors.yml` switches to opt-out mode and would
  orphan the generated key; a live deployment changes modes only by rebuild.

## Commands

```sh
./green build
./green create --dry-run
./green create
./green delete
```

`./red` and `./blue` run the same verbs against the same state; never run two
colours concurrently. Build and dry-run require no credentials and never touch
`~/.ssh`. Never export `COLORS_PAR_PROFILE`. Keep
`compute-prevent-destroy: true`; deletion requires separate authorization and
a one-run `COLORS_PAR_COMPUTE_PREVENT_DESTROY=false` override.

## Provider switching is a rebuild

Every real `create` and `delete` reads the recorded `params.provider` from
state before validating provider credentials and refuses when it differs from
`provider-compute`. To move this profile to another provider: `delete` on the
recorded provider, then edit `provider-compute` and `create`. Never edit the
provider on a profile with a machine in state.

## The ssh alias

Convergence writes a `~/.ssh/config` block per the workspace SSH Config
Standard, so `ssh posthog-vultr` reaches the host with no address, user or
`-i` flag. Delete removes the block before destroying the instance. The block
is inserted at the top of `~/.ssh/config`; if that file ever grows an option
above its first `Host` line, `create` refuses rather than capturing a global
setting into this deployment's stanza.

## Sizing

PostHog is a multi-tier suite: Postgres, ClickHouse with embedded Keeper,
Redpanda, Redis, Temporal and three application processes. 8 GiB wedged the
DigitalOcean machine hard enough that sshd could not fork; `vc2-6c-16gb` is
Vultr's smallest 16 GiB plan and the floor here.

## Documentation

`index.html` is this repository's landing page and carries two analytics tags:
GA4 measurement ID `G-4VKP1WY4QJ`, whose explicit `page_title` must exactly
equal the decoded HTML `<title>` and stay distinct and stable so one Analytics
property can separate repositories, and the self-hosted Rybbit snippet
`<script src="https://rybbit.getcolors.ai/api/script.js" data-site-id="9fb9c41a6d49" defer></script>`,
which shares one site ID across every page because `getcolors.github.io/<repo>/`
paths already encode the repository. Never add one tag without the other.

`verification.md` records the live verification this deployment passed:
commands, timestamps, provider, plan and pin, and each gate's outcome. It
carries no address, key id or token.

## Git

The root `green`, `red` and `blue` are copies, not symlinks. After a Package
Skill update run `npx skills update -p -y` and copy
`.agents/skills/package-posthog-<colour>/<colour>` over each. Never hand-edit
a SHA.

Work on the current branch. Do not commit or push unless explicitly asked.
