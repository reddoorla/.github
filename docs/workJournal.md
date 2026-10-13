# reddoorla/.github — Work Journal

Running log of work on the org's shared CI and Renovate configuration: what was
done, why, and where it landed. Chronological — newest entry at the bottom.

The convention is in [CLAUDE.md](../CLAUDE.md) under "The work journal". In
short: every working session appends a dated entry, prose over bullets, why over
what, and history is never edited to be right — a later entry corrects an
earlier one and says so.

---

## 2026-09-05 — Journal opened, and 33 commits of history summarised rather than reconstructed (`chore/work-journal`)

The journal starts today, so this first entry is a **backfill**: a deliberately
coarse summary written from the commit log and the merged PR list, not from
memory. Detail below this line is trustworthy; detail above it is not, and
nothing here should be cited as though someone wrote it down at the time. The
commit log and the PR bodies remain the record for anything before 2026-09-05.

**What this repo is.** GitHub's org-level `.github` repository for `reddoorla`,
holding two things every other repo in the fleet consumes: reusable workflows
(`ci.yml`, `prismic-models.yml`) that site repos call by SHA, and
`renovate-config.json`, the org Renovate preset that every repo extends as
`github>reddoorla/.github:renovate-config`. It has no application code, no
`package.json` and nothing to build — it is YAML and JSON that runs in other
people's repositories.

**The eras, roughly.** All 33 commits are 2026, from `v1.0.0` on 2026-06-08 to
here: 10 in June, 5 in July, 17 in August, 1 in September. The split by file
says what the work actually was — **20 commits touch `renovate-config.json`**
against 9 on `ci.yml` and 3 on `renovate.yml`. June built the thing: the
reusable CI workflow, the Netlify deploy-preview PR comment, lockfile
maintenance and OSV alerts. July is small in count and large in consequence —
after a bulk `gh pr merge --auto` sweep on 2026-07-26 armed GitHub's platform
auto-merge across 8 repos and two `actions/checkout` majors merged with zero
reviews, #11 and #12 routed every merge back through Renovate's own run with
`platformAutomerge: false`, which is why this repo's cron is now the *merge*
cadence and not just the PR-creation cadence. August is almost entirely version
holds learned from a weekly batch turning fleet CI red: TypeScript below 7,
`cookie` below 2, `@libsql/client` at 0.8.x, pnpm override entries disabled
outright, and a `vite-plugin-svelte` hold added and then removed three weeks
later once the vite-8 wave landed. #28 and #29 belong together — #28 fixed a
real deadlock between the Monday-only schedule and the 1-day age gate, and #29
corrected #28's own claim that 18 fleet PRs had frozen when 16 of them were a
packageRule working as designed. That correction went into the config's
`description` array because there was nowhere chronological to put it, which is
roughly the gap this file closes.

**State as of this entry.** `main` at `8f9852c`, tree clean, nothing in flight —
all 31 PRs ever opened here are merged, none open. Tagged through `v1.4.1`.
Consumers pin by commit: 17 site repos are on `ci.yml@v1.4.1`, 3 are still on
`v1.2.0` and 2 on `v1.3.0`, and 8 call `prismic-models.yml@v1.4.0`. One
leftover: `origin/renovate/all-minor-patch` carries an unmerged
`renovatebot/github-action` v46.2.4 bump from 2026-08-31 with no PR open,
which is what `prCreation: not-pending` looks like from the outside — the
branch waits for a Monday window in which it can both open and merge.

**What changed today.** `CLAUDE.md` and this file, nothing else.

## 2026-09-15 — The reusable lint step now has a known-bad input, and a guard so the next repo cannot rejoin the blind spot (#818)

`ci.yml` has always run `pnpm exec prettier --check .` bare. Prettier loads
`prettier-plugin-svelte` only when a **config file** declares it, so in a repo
with no `.prettierrc*`, no `prettier.config.*` and no `prettier` key, that step
parses zero `.svelte` files and exits 0. It is the CLAUDE.md failure mode
exactly: a check that had only ever passed, passing because it was reading
nothing. Four repos were in that state at once — `reddoor-starter-blux` (82
`.svelte`), `the-tower-burbank` (80), `canvas-starter` (68) and
`composition-hospitality` (64) — and `reddoor-starter-blux` is the template
every `--track blux` site is cloned from, so the hole was being copied forward.

Measured on `the-tower-burbank` before any fix, with one deliberately mangled
`Nav.svelte`: `pnpm exec prettier --check .` printed "All matched files use
Prettier code style!" and named the file **zero** times, while the repo's own
`pnpm lint` — which passed `--plugin prettier-plugin-svelte` on the command
line — flagged it. The CLI idiom was carrying the whole fleet's Svelte
formatting, and CI never shared it.

Two things landed here. `tests/prettier-svelte/` holds two trees with a
byte-identical mis-formatted `.svelte` file, and `validate.yml` asserts they
produce **opposite** outcomes: with the shared config the lint command must fail
and name `src/Mangled.svelte`; with no config it must pass. The second
assertion is the interesting one — it pins the blind spot deliberately, so if a
future Prettier starts parsing `.svelte` without a configured plugin, the test
fails and tells us the guard can be retired rather than silently protecting
nothing.

The guard itself is an inline step in `ci.yml` that refuses a repo with
`.svelte` files and no Prettier config. It is inline rather than a composite
action or a second checkout so that nothing floats: a fleet repo pinned to a
tag executes exactly the bytes of that tag. To keep the test honest about
*which* bytes, `validate.yml` extracts the shell between two markers in
`ci.yml` and runs that, so the tested artifact and the shipped artifact cannot
drift apart.

The guard reaches a repo only when that repo moves its pin, so nothing changes
for the fleet until each site bumps its `uses:` line. That is deliberate — this
tag should roll out behind the config PRs, not ahead of them, or the guard would
red every one of the four repos before their fix merged.

## 2026-10-12 — The staging flow gets its own opt-in preset, renovate-staging-flow (patch from a reddoor-maintenance cloud session)

The operator answered Operator decisions 113 (reddoor-maintenance#545): patches and minors target `main` and fast-track, because "if I have to PR and merge for every patch update it pretty much defeats the point of renovate". Majors target `staging` and reach production only through his promotion PR. This file is that rule. It is a separate preset rather than a change to `renovate-config`, because its rules disable majors on `main`. On a repo with no `staging` branch that would leave majors with nowhere to go. Renovate only warns and skips a base branch that does not exist (`workers/repository/process/index.js`, "Base branch does not exist - skipping"), so the absent branch would never be loud.

It was proven before it shipped, on vida-legacy-foundation, by a draft PR that ran this repo's Renovate (44.151.2, the reddoor-renovate token) with `dryRun: full` in two arms (vida-legacy-foundation#99, closed). With the preset, the patch and minor group would open as `renovate/main-all-minor-patch` with `labels: ["fast-track"]`. Kit 3, adapter-netlify 7 and the vitest major, lock-file maintenance (`labels: []`) and the `actions/setup-node` digest all went to `staging`. The control arm, the fleet preset alone, saw only `main` and no `fast-track` anywhere. A cloud session cannot run this on its own: Renovate's GitHub platform calls GraphQL in `initRepo`, and the session proxy refuses GraphQL. The local platform skips base branches entirely (`platform !== "local"`).

Three things read from Renovate's source shaped it:
- **Branch names.** Two base branches prefix every branch name with the base (`renovate/main-…`, `renovate/staging-…`). So the first live run on a repo autocloses its open Renovate PRs and reopens them under the new names; the dry run listed vida's #80 and #94–#97.
- **Grouped labels.** A grouped PR takes the union of its upgrades' `addLabels` (`workers/repository/updates/generate.js`). That is why one rule on `main` patch/minor is enough to mark the group.
- **Vulnerability alerts.** Each alert becomes a packageRule with `force: { enabled: true }` (`workers/repository/init/vulnerability.js`), which overrides this preset's `enabled: false`. The dry run showed vida's sharp fixes on both bases, and the one into `main`, a minor, carries the label.

The label is how `main-gate`, the required check that goes on each flow repo's `main`, tells a patch or minor from anything else. That gate passes only:
- a PR from `staging`, or
- a PR by `reddoor-renovate[bot]` on a `renovate/` branch where every commit is the bot's and the `fast-track` label was last added by the bot.

`validate.yml` now holds this file to that shape, and each of its six assertions was turned red by a mutation before this landed:
- `baseBranchPatterns` is exactly `[main, staging]`;
- no rule sets `automerge` (`renovate-config` owns it, with `platformAutomerge: false`);
- each of Renovate's ten update types targets exactly one base, and only `patch` and `minor` target `main`;
- exactly one rule adds `fast-track`, and it is bound to `main` and to `patch`/`minor`.

Not changed here: the last packageRule in `renovate-config` (reddoor-website's unprotected `staging`) says to delete it when that `staging` gains a required check. It has not gained one yet, so the rule stays.
