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
