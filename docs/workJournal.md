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

## 2026-10-22 — Grouped minor/patch PRs open on the run that pushes them, and the never-automerge packages group alone (patch from reddoorla/reddoor-maintenance#1689)

This is option 1 of the reddoor-workspace decision `decisions/2026-10-21-2218-renovate-feature-drought.md`: 19 of 25 repos had no grouped minor/patch PR opened from 2026-09-22 to 2026-10-21. The mechanism and the two-arm proof (reddoorla/reddoor-starter#197, reddoorla/canvas-starter#36) are in the two packageRules dated 2026-10-21 and in the workspace's `audits/2026-10-21-renovate-feature-drought/addendum-2026-10-21-proof.md`, so they are not restated here.

It was built and proven in a cloud session but landed from the laptop, as a `git am` of the patch stored in reddoorla/reddoor-maintenance#1689, because a cloud session cannot attach a repository whose name starts with "." and this one does. The patch applied cleanly on `2de84b3` and touches only `renovate-config.json` and `validate.yml`.

Before the push, both `renovate` steps, extracted from the patched `validate.yml` and run under Node 24.16.0, printed OK. Then each path of the two new guards was turned red by a mutation of its own, and all four exited 1:
- the patched validator against `main`'s unpatched config (`2de84b3`): "no unscoped minor/patch rule with prCreation:immediate";
- the `immediate` rule scoped with `matchPackageNames`: the same error;
- a later minor/patch rule with `prCreation: not-pending`, scoped by `matchDepTypes`: the same error, because the last such rule decides;
- the `held-minor-patch` group removed: "never-automerge packages [...] have no minor/patch group of their own".

Nothing reaches a site until its next run inside the window, Monday 2026-10-26 before 18:00Z (11am PT), when it resolves this preset at runtime. reddoor-starter, canvas-starter, medical-solutions-of-texas and hedloc all extend `reddoorla/.github:renovate-config` with no `#` version, so no pin moves for them; the rest of the fleet was not checked. Expect the wave the preset's 2026-10-21 note describes: two grouped PRs per repo on the first in-window run, throttled by `prHourlyLimit`.

## 2026-10-24 — Majors open on the run that pushes them, and lock-file maintenance takes the first hourly PR slot (patch from reddoorla/reddoor-maintenance#1688)

The follow-up to the drought patch (#39). The operator picked it in reddoor-workspace `decisions/2026-10-24-1413-renovate-majors-and-pr-slots-1688.md`, and then narrowed it in `decisions/2026-10-24-1436-renovate-1688-digests-and-slots.md`. The first decision asked for `prCreation: immediate` on majors and digests, and `prPriority` on lock-file maintenance. The proof arms on reddoorla/reddoor-starter#208 found the digest half rested on a wrong premise, so the operator dropped digests (option (a), Sat 10-24 at about 8:00am PT).

The belief corrected on contact: #1688 said a new digest branch "gets rebased every Monday and opens its PR only in a week when a second in-window run pushes nothing", citing `renovate/actions-setup-node-digest` as "branch present and no PR" in a 10-21 dry run. Neither half is true on this preset. A digest has no release timestamp, so under `minimumReleaseAge: "1 day"` (Renovate's default `timestamp-required`) its stability check is pending forever, and `prCreation: not-pending` never creates the branch: `Skipping branch creation due to internal status checks not met`, in every control arm. The dry run's "branch present" was its `Processing N branches:` line, which lists every candidate branch whether or not it exists. Made immediate, the digest does open (reddoor-starter#209, #211), behind `renovate/stability-days: pending — Updates have not met minimum release age requirement`, which never clears. And because Renovate sorts digest ahead of patch and minor, the round-1 arm with both rules as first decided opened lock-file maintenance (#210) and then the digest (#211), and neither grouped PR got a branch. That is the drought again, for the automerge-eligible group.

`prPriority` was the unverified half of #1688, so it was read before relying on it. `lib/workers/repository/process/sort.ts` orders branches by vulnerability alert, then `prPriority`, then update type in the order pin, digest, patch, minor, major, lockFileMaintenance. Once the hour's PR count reaches `prHourlyLimit` (Renovate's default of 2), every later branch logs `Hourly PRs limit reached` and is not even created. So until now lock-file maintenance always came last. The sort, limit and branch files are identical in 44.176.1, 44.181.2 and 44.181.3; the arms ran 44.181.2 (round 1, 7:32am PT) and 44.181.3 (round 2, 8:02am PT).

Round 2, the final shape, real runs on reddoor-starter under `probe/1688-r2-*`:
- with only majors and digests enabled (the digest is never branched) and no hourly limit, the control logged `Branch created` then `Branch status pending, current sha: 1e8c08b…` for both majors and opened nothing; with the rule, `Branch created` then `PR created` for both (#217, #218);
- with `prHourlyLimit: 2`, the control opened `all-minor-patch` (#219) and `held-minor-patch` (#220), and lock-file maintenance logged `Hourly PRs limit reached`; with the rules, lock-file maintenance opened first (#216) and `all-minor-patch` second (#221).

Every arm PR was a draft and is closed unmerged; the `probe/1688-*` branches stay, because the cloud proxy refuses branch deletes.

What `prHourlyLimit: 2` still costs, stated so it is not mistaken for a fix: in a week when both grouped branches refill, the Monday run opens lock-file maintenance and one grouped PR, and the other grouped PR and any new major wait. The hourly count resets each clock hour, so a second in-window run would open them the same day; but the 12:00Z cron almost always starts after the window closes (17:51Z to 21:12Z, measured 10-21), so in practice they usually wait for the next Monday with a free slot. Raising the limit was option (c), not chosen.

`validate.yml` now refuses three things. Under `not-pending`, an unscoped rule must set `prCreation: immediate` for `major`, and no later rule reaching `major` (a rule with no `matchUpdateTypes` counts) may set anything else. `prCreation: immediate` must not reach `digest`, whether top-level, in a `digest` block, or by a rule naming `digest` or naming no update type. And with lock-file maintenance enabled, the last `prPriority` rule reaching it (a typeless one counts) must be unscoped, match `lockFileMaintenance` alone, and give it the strictly highest `prPriority`. Nine mutations turned the step red, each with its own error: the majors rule removed, scoped, or overridden by a later scoped `not-pending` rule; `digest` added to it; the priority rule removed, set to 0, outranked, scoped, or moved inside `lockFileMaintenance` (where Renovate does not accept it). The unchanged patched preset passed, and `main`'s preset at `a726791` failed on the majors guard. Review round 1 (session model) found the guards too narrow (a priority rule that also matched minor and patch tied with the groups and passed; a typeless `prCreation` override and three other routes to an immediate digest passed), and found the first text calling the next-Monday wait certain when it is only usual; both were fixed before this landed, with six more mutations. Round 2 found one more: a later typeless or scoped `prPriority: -1` tied lock-file maintenance with the groups again and passed; that and a false refusal of a harmless later scoped `immediate` major rule were fixed too, with the mutations listed in reddoor-maintenance#1712.

## 2026-11-09 — Node held below 26 in the org preset until AWS Lambda's nodejs26.x is GA (patch from reddoor-maintenance P1-105)

Renovate opened "update node.js to v26" PRs on eight fleet repos and reddoor-md-pdf in the 11-02 and 11-09 windows. The operator holds Node 26 until AWS lists Lambda `nodejs26.x` as GA (reddoor-workspace `decisions/2026-11-04-1809-erp-node26-functions-stay-on-24.md`). The reason is a type gap. Netlify runs server functions on the build's Node only when Lambda supports that major for production, and otherwise falls back to `nodejs24.x`. So a Node 26 build with `@types/node` 26 typechecks Node-26-only APIs that then throw in production. On 11-09 at 11:31am PT, one of those PRs (medical-solutions-of-texas#114) was merged into MSOT `staging` by hand, while AWS still listed the runtime as a public preview. The operator confirmed the hold still stands (`decisions/2026-11-09-1934-p1-105-msot-node26-merged-against-hold.md`, answer (a)). MSOT's revert is a separate task, and this preset rule does not undo it, because Renovate does not downgrade.

The rule is one `allowedVersions: "<26.0.0"` on `matchDepNames: ["node", "@types/node"]`, with no `matchManagers`. A fixture repo carried Node in every place the fleet might: `.nvmrc`, `.node-version`, `.tool-versions`, a `node` Dockerfile, `engines.node`, `@types/node` and a setup-node `node-version`. Renovate 44.199.2 extracted every one of them as depName `node` (`@types/node` for the types). The managers are `nvm`, not `nvmrc` as first written; `nodenv`, `asdf`, `dockerfile` and `npm`; and `github-actions`, with packageName `actions/node-versions`. Listing no manager is what keeps a new place from slipping past the rule.

Measured with `--platform=local --dry-run=lookup` on that fixture:

| Arm | Node proposals |
|---|---|
| Control (this preset before the patch) | 26.13.0 on nvm, nodenv, asdf and dockerfile; `@types/node` ^26.0.0 |
| Patched | none |
| Cap set to `<27` | 26 on every one of them again |
| Patched, fixture moved back to 24.10.0 | minor 24.22.2 on all four Node files |

The last arm is the check that the cap does not block the Node 24 patches and minors the fleet still needs.

The github-actions leg cannot be looked up from a cloud session, because its datasource goes through GraphQL and the proxy refuses GraphQL. It was checked instead with Renovate's own `applyPackageRules` and node versioning on the extracted dep. Patched, `allowedVersions` is `<26.0.0` and 26.13.0 is refused. With the rule scoped to the other five managers, it is uncapped.

`validate.yml` now asserts the hold for each [manager, dep, package] leg. The last `allowedVersions` rule that can reach the leg must be matched only by manager, dep or package names that all name it, and must cap below 26. Four mutations each turned the step red, naming the leg:
- github-actions dropped from a `matchManagers`
- the cap set to `<27`
- `@types/node` dropped
- the rule removed

The unchanged patched preset passed, and so did `renovate-config-validator --strict`, as both global and repo config.

A belief corrected on contact: the first draft capped at `<25`, on the theory that `<26` would let Renovate propose `@types/node` 25 and Node 25. It does not. Under `<26`, the dry run proposed neither, although `@types/node` 25.9.9 exists on npm. Renovate looks up `@types/node` with node versioning (`workarounds:typesNodeVersioning`), and that versioning treats an odd, non-LTS major as unstable.

reddoor-maintenance's deps audit still reads `typescript` as the one held cap (`capsFromPreset` gives `[["typescript","<7.0.0"]]` before and after). The new rule names its packages exactly, so it cannot read as touching typescript.

An adversarial review (round 1) found only minors and nits, and they are folded in here. A registry-prefixed image (`FROM docker.io/library/node:24.10.0`, or `cimg/node`) has the image as written for its depName, and is still offered 26. No reddoorla repo uses one (code search, 11-09; reddoor-md-pdf's Dockerfile is `node:22.22-slim`). Covering one would take a regex `matchPackageNames`, which reddoor-maintenance's deps audit reads as possibly reaching `typescript`, so the description names the gap instead. The description had also credited `group:nodeJs` with putting `@types/node` on the Node PR. In fact Renovate strips `@types/` when naming branches, so both land on `renovate/node-26.x`. The guard had passed four presets it should refuse:
- a capitalised `matchDepNames: ["Node"]` rule with a high cap (Renovate matches names case-insensitively);
- a later uncapped `mise` rule;
- a later uncapped `docker-compose` rule;
- a cap of `<1`, which would also block Node 24.

It now compares names case-insensitively, adds the mise, docker-compose and workflow `container:` legs, and accepts only `<25` or `<26`. All four fail now. The guard also moved above the lock-file `prPriority` comment, which it had split from its code.

Autoclose has a caveat. Renovate closes a branch it no longer proposes only when no one else has committed to it. If someone has, it retitles the PR "abandoned" and leaves it open.

To lift the hold: when AWS's runtimes page lists `nodejs26.x` without the preview note, delete the rule and the `validate.yml` block together, then follow P1-99's per-site recipe.
