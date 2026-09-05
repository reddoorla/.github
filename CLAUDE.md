# CLAUDE.md

This is GitHub's org-level `.github` repository for `reddoorla`. It holds the
reusable workflows every fleet repo calls (`.github/workflows/ci.yml`,
`prismic-models.yml`) and `renovate-config.json`, the org Renovate preset that
every repo extends as `github>reddoorla/.github:renovate-config`. There is no
application code, no `package.json` and nothing to build or test locally — it is
YAML and JSON that executes in other repositories.

Three things worth knowing before editing:

- **A change here ships fleet-wide, but only to repos that move their pin.**
  Site repos pin `uses: reddoorla/.github/.github/workflows/ci.yml@<40-hex>`
  with the tag in a trailing comment, so a change is not live anywhere until it
  is tagged and the call sites are updated. Sites are currently spread across
  several tags.
- **`validate.yml` is the only CI this repo has**, and it runs on every PR: it
  fails on any third-party `uses:` that is not digest-pinned to a 40-hex commit,
  and it parses `renovate-config.json` to assert that every automerge rule
  carries `platformAutomerge: false`. Both guards exist because of real
  incidents; do not route around them.
- **`prismic-models.yml` is not authored here.** Its source of truth is
  `reddoor-maintenance`, where it is tested against the CLI it calls; this copy
  is published from there. Its own header says so. Edit it upstream.

## The work journal

**Every working session appends a dated entry to `docs/workJournal.md`** — what
was done and **why**, newest at the bottom, never corrected in place. Write it
as the last act of the session, not the first act of the next one.

The journal is the history of executing the build. Code says what the system
does now; the journal says what it used to do, what it cost to change, and
which beliefs turned out to be wrong. Nearly everything expensive to rediscover
lives there and nowhere else.

An entry is headed with the date, a short title, and where it landed:

```markdown
## 2026-09-04 — Both runway stages render their final frame without JS (#51, `ce46ae0`)
```

Then prose — not a bullet list of file names, which the diff already tells you.
What to put in, in rough order of value:

- **Why, over what.** The reason a thing was done survives; the diff does not
  need restating.
- **Measured numbers, exactly.** "The comp's open mask is 2696×2352 on an 860px
  band — 2.735× the band's height, so a 390×664 phone needs ~534%" is worth
  keeping. "Fixed the hero on mobile" is not.
- **Defects, named.** What broke, what it looked like, and what made it
  invisible until it wasn't.
- **What was tried and abandoned**, and what it would take to revive it. A dead
  end nobody wrote down gets walked twice.
- **Beliefs corrected on contact.** The design assumption that turned out false
  is usually the most valuable line in the entry.
- **Honest accounting.** If a win came from somewhere other than the change
  that claimed it, say so — that is exactly what someone will otherwise
  over-invest in next.

**History is never edited to be right.** An entry that stops being true is not
rewritten; a later entry corrects it, and says which one it corrects. The
journal is a record of what was believed at the time, and that record is most
useful precisely where it was wrong. Fixing the past in place destroys the only
evidence of how the mistake was made.

The one edit an old entry may take is a **forward pointer**: one line directly
under its heading naming the entry that overturned it — `> Superseded in part by
2026-10-14 — <that entry's title>.` It asserts nothing new and retracts nothing,
so the record of what was believed survives whole; it only stops a reader who
lands on the old paragraph from leaving with the old answer. Without it the rule
above is half a mechanism: the correction exists at the bottom of the file, and
nothing points to it from where a reader actually arrives.

If a session produced nothing worth an entry, that is itself worth one line.
