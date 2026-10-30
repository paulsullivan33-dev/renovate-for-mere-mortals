# Your first dry run

**The outcome of this chapter:** you can run Renovate
against a project in a way that writes nothing at all, and
read its output well enough to say, for every dependency it
found: what it would update, to what, and on which branch.

A **dry run** is a rehearsal: Renovate does the reading
and the looking-up, and skips the writing. For a first
encounter with a robot that edits code, that is exactly the
right shape. Renovate offers several dry-run levels; the
one that matters here is built into the `local` platform
(chapter 2), which works on a plain directory and cannot
create branches or pull requests at all.

Three honest limits of the local platform before we start,
from the project's documentation: it runs in the current
directory only; it defaults to `dryRun=lookup`, and other
levels such as `full` fall back to `lookup` rather than
being honoured; and presets borrowed from other people's
accounts (written like `github>somebody/shared-config`)
cannot be fetched, because there is no platform account to
fetch them with. None of that stops it being the best
first step in this guide.

## The fixture

The lab ran against an invented project, the recipe box,
which lives in `renovate-lab/fixture/` of this guide's
repository so you can repeat every command. It is three
files that matter:

- `package.json` naming two dependencies, pinned at old
  versions on purpose: `chalk` 4.1.2 (July 2021) and
  `lodash` 4.17.20 (August 2020).
- A `Dockerfile` standing on `node:18.20.4-alpine`, a
  Node.js 18 image with a long line of successors.
- Later in the lab, a `package-lock.json` and a
  `renovate.json` join them; the first runs below are
  without either.

## Step one: extract only

`--dry-run=extract` tells Renovate to stop after the
reading stage. It changes nothing and needs no network:

```console
$ cd fixture
$ LOG_LEVEL=debug renovate --platform=local --dry-run=extract
```

Renovate's normal voice is quiet, so the lab turned on
`LOG_LEVEL=debug`; the full transcript is t01. The part
worth framing:

```
INFO: Dependency extraction complete
      "stats": {
        "managers": {
          "dockerfile": {"fileCount": 1, "depCount": 1},
          "npm": {"fileCount": 1, "depCount": 2}
        },
        "total": {"fileCount": 2, "depCount": 3}
      }
```

Two files understood, three dependencies found, each
manager credited with its own. The block after it (kept in
full in the transcript) lists the three dependency records
— `node` 18.20.4-alpine from the Dockerfile, `chalk` 4.1.2
and `lodash` 4.17.20 from the package file — in the shape
chapter 2 showed you. If this stage finds fewer
dependencies than you know your project has, stop here:
every later stage can only work with what extraction
found, and chapter 5 is about how extraction decides.

Your check: the run ends `INFO: Repository finished`, the
counts match your project, and nothing in your directory
has changed — `git status` stays clean.

## Step two: look everything up

Leave off the `--dry-run` flag. On the local platform the
default dry-run level is `lookup`, which adds the second
stage: every datasource is asked what versions exist now.

```console
$ LOG_LEVEL=debug renovate --platform=local
```

The lab's first lookup run (transcript t02) produced, in
about twelve seconds, the whole story of a real project
review. For the two npm dependencies, the registry answered
in full:

- `chalk` 4.1.2 → **6.0.1**, an update of type `major`,
  flagged breaking, on would-be branch `renovate/chalk-6.x`.
  The record includes when the current version was released
  (the lab's log puts it 1,918 days old) and when the new
  one was (33 days before the run) — Renovate reads
  timestamps as well as numbers, which chapter 6 uses.
- `lodash` 4.17.20 → **4.18.1**, type `minor`, not flagged
  breaking, branch `renovate/lodash-4.x`.

The summary lines are the quickest health check a project
can get:

```
DEBUG: dependencyStatus: {"outdated": 2, "total": 3}
DEBUG: 2 flattened updates found: chalk, lodash
DEBUG: Returning 2 branch(es)
```

And one number worth meeting early: Renovate totals how
far behind a project is in **lib-years** — the summed age
of the gaps, as if one library were that many years stale.
This fixture scored 10.79 npm lib-years. It is a vanity
metric with a serious use: run monthly, it tells you
whether you are keeping up or sliding.

## The third dependency: a lookup that failed

The Dockerfile's `node` dependency got no update in that
run, and the reason is in the log, kept in the transcript
on purpose:

```
DEBUG: GET https://auth.docker.io/token?... = (code=ECONNRESET, ...)
WARN: docker registry connection failure
      "registryHost": "https://index.docker.io",
      "dockerRepository": "library/node",
      ...
      "message": "socket hang up"
```

Docker Hub's sign-in service reset the connection, so the
docker datasource came back empty-handed, and Renovate
recorded a warning on that one dependency (`node: socket
hang up`) and carried on with the other two. Three lessons
live here, and they are worth more than the update would
have been:

1. **A failed lookup is not a failed run.** Datasources
   fail independently. One registry being unreachable costs
   you that dependency's updates, not the run.
2. **"No update found" and "could not look" are different
   answers**, and Renovate's output distinguishes them —
   the warning rides along on the dependency itself. On a
   real platform that warning is exactly the sort of thing
   the Dependency Dashboard (chapter 8) surfaces.
3. **Registries are just web services.** They have bad
   minutes. The lab ran the same command later with a
   config file added (transcript t04) and the same lookup
   succeeded: `node` had two updates waiting, 18.20.8-alpine
   (a patch) and 26.13.0-alpine (a major). Chapter 6 tells
   that half of the story.

One more line from that first run is worth knowing before
it surprises you in your own: with no `renovate.json` in
the fixture, the log says the repository "is not
onboarded" and notes it would create an onboarding branch.
That is chapter 8's subject. For the dry run it changes
nothing — the lookups above are the same ones an onboarded
project would get.

You have now seen the robot read and think. The next two
chapters open up those two stages — the reading, then the
asking — before chapter 7 puts you in charge of both.
