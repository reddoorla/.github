# prettier-svelte fixtures

Two trees holding a byte-identical, deliberately mis-formatted `.svelte` file.

`validate.yml` runs the reusable workflow's own lint command against both and
asserts opposite outcomes:

- `with-config/` has the fleet's shared `.prettierrc.json`, so the command MUST
  fail and name `src/Mangled.svelte`. That is the gate biting.
- `no-config/` has no Prettier configuration, so the command MUST pass. That is
  the blind spot from reddoorla/reddoor-maintenance#818, pinned deliberately: if
  a future Prettier starts parsing `.svelte` without a configured plugin, this
  assertion fails and tells us the guard can be retired.

Do not "fix" the formatting of these files. Their mangled state is the input.
