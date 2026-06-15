# AGENTS.md

Secret-handling boundary for any agent (or person) working in this repo. This
repo is the live `/config` of the Home Assistant VM; it sits next to real
household secrets, so the boundary is strict.

## Hard boundary

Do **not** generate, decrypt, view, copy, transform, rotate, migrate, or print
secrets unless the operator explicitly requests that exact action.

Secrets in this repo include:

- `secrets.yaml`
- everything under `.storage/` (registry, tokens, credentials)
- the Zigbee network key and MQTT credentials — `zigbee2mqtt/`,
  `coordinator_backup.json`, any `network_key`
- long-lived access tokens, `*.key`, `*.pem`, `.google.token`

All of the above are gitignored.

## Rules

- The **`.gitignore` is a safety control.** Never weaken, remove, or override it.
  Never `git add -f` a gitignored path.
- **Before every commit**, confirm nothing sensitive is staged:
  ```sh
  git ls-files | grep -E "secrets.yaml|\.storage|zigbee2mqtt|coordinator_backup|network_key" \
    && echo "STOP — sensitive file staged" || echo "clean"
  ```
- Reference secrets with HA's `!secret <name>` syntax. Never inline a real value.
- If a task seems to need secret access, **stop and ask** — name the file, the
  exact fields, and whether you may only validate vs. modify.

## Allowed without asking

- Author/edit behavior YAML: automations, scripts, scenes, templates, packages,
  dashboards, custom blueprints.
- Create placeholder/templated config referencing `!secret` keys, and document
  required secret key names.
- Verify that files/keys exist without printing their contents.

## Authority

This file is the local summary. The full policy is the homelab monorepo's
`~/code/homelab/AGENTS.md` and `~/code/homelab/docs/secret-handling-boundary.md`;
defer to them on any conflict.
