# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repo.

## What this repo is

The live `/config` of the Home Assistant VM (HAOS, `10.0.0.90`) — **authoritative
for what Home Assistant actually runs.** It is the Phase 10 home-automation slice
of a family homelab. The control-plane strategy, domain glossary, and decisions
live in the **homelab monorepo** (`~/code/homelab`), not here:

- Glossary: `~/code/homelab/CONTEXT.md`
- Strategy: `~/code/homelab/services/home-automation/README.md`
- Decisions: `~/code/homelab/docs/adr/0003`, `0005` (Zigbee2MQTT over ZHA),
  `0006` (config-as-code is VM-canonical)

Read those for the domain language and use it exactly (Awareness Capability,
Actuation Authority, Promotion, Family-Critical Service, Break-Glass Recovery).

## Secret-handling boundary (hard rule)

Do **not** read, print, decrypt, copy, transform, rotate, migrate, or commit
secrets unless the operator explicitly requests that exact action. In this repo
secrets include: `secrets.yaml`, anything under `.storage/`, the Zigbee network
key and MQTT credentials (`zigbee2mqtt/`, `coordinator_backup.json`), long-lived
access tokens, `*.key`, `*.pem`. These are gitignored.

The **`.gitignore` is a load-bearing safety control — never weaken it, never
`git add -f` a gitignored path.** Before any commit, verify nothing sensitive is
staged:

```sh
git ls-files | grep -E "secrets.yaml|\.storage|zigbee2mqtt|coordinator_backup|network_key" \
  && echo "STOP — sensitive file staged" || echo "clean"
```

Author config that *references* secrets with HA's `!secret <name>` syntax; never
inline a real secret value. Full policy: `~/code/homelab/AGENTS.md` and
`~/code/homelab/docs/secret-handling-boundary.md`. See also `./AGENTS.md`.

## How this repo is operated (Model B, Option A)

The **VM is canonical.** Home Assistant config changes happen on the VM (the HA
UI and the code-server add-on both write to `/config`). This workstation clone is
for **authoring assistance only** — drafting chunky artifacts (packages,
dashboards, templates) that are easier to write as text.

Workflow:

1. Author/edit the behavior file here.
2. `git push` (after the secret check above).
3. On the VM (code-server terminal): `git pull` into `/config`.
4. HA **Developer Tools → YAML → Check Configuration**, then reload
   (or restart for new packages/helpers).

Soft rule to avoid two-writer conflicts: a file you author here, treat as code —
don't also edit it in the HA UI. Quick UI tweaks → pull them back before working
here. When in doubt, `git pull` first.

There is no local Home Assistant to lint against, so YAML must be correct by
inspection; the VM's Check Configuration is the validation gate.

## What is tracked vs not

- **Tracked (behavior, benefits from review):** `configuration.yaml`,
  `automations.yaml`, `scripts.yaml`, `scenes.yaml`, `packages/`, dashboards,
  custom blueprints.
- **Not tracked (device layer, recovered from HA Backup — ADR 0006):**
  `.storage/`, `zigbee2mqtt/` pairings, the recorder DB, caches, `secrets.yaml`.

## Governing rule for changes

An **Awareness Capability** (HA reads/reports state) is in-scope by default. Every
**Actuation Authority** (HA changes physical state — lights, fan, thermostat,
alarm) ships observe-first and needs an explicit grant plus a documented manual
fallback before it is trusted. If HA fails, the house must still work.
