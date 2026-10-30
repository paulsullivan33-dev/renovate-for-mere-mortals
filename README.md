# Renovate for Mere Mortals

Every project built from other people's
code slowly goes stale. The libraries it
borrows release new versions; the base
images it stands on release new versions;
and somebody — usually you, late, by hand —
has to notice, edit the version numbers,
and hope the tests still pass. **Renovate**
is a robot that does the noticing and the
editing. It reads your project, works out
everything it depends on, checks each
dependency's home registry for newer
versions, and opens a pull request for
each update it finds. You stay in charge:
nothing merges unless you — or a rule you
wrote — say so.

This guide explains how Renovate reads a
project, where it gets its facts about new
versions, and how to tell it exactly how
you want your updates: grouped or separate,
scheduled or immediate, merged by hand or
— carefully — by itself.

The lab that wrote this guide ran the real
thing: **Renovate 44.198.2**, installed
from npm, against a tiny invented project
with three deliberately old dependencies —
two JavaScript libraries in a
`package.json` and a Node.js base image in
a `Dockerfile`. Renovate found all three,
looked the JavaScript libraries up in the
npm registry, and reported real updates: a
major release five years newer than the
pinned one, and a quiet minor one. It read
the project's lock file, applied the
project's own configuration file — grouping
the updates the way the file asked — and
named the branches it would have created.
Its configuration validator was run on a
good config and a deliberately broken one,
and caught three of the four planted
mistakes.

What the lab could not do, it did not
fake. The lab has no GitHub account to
lend the robot, so everything past the
point where Renovate would talk to a real
hosting platform — the onboarding pull
request arriving, the Dependency Dashboard
issue, a pull request actually opening, an
automerge actually merging — is taught
from the project's documentation and
labelled as such wherever it appears.
Renovate's local dry-run mode exists
precisely so you can see its thinking
before any of that happens, and that mode
is what the lab ran. `criteria.md` keeps
the full ledger of what was run and what
was only read.

## What this guide covers

- [**What Renovate is**](what-renovate-is.md) —
  the stale-dependency problem, the robot
  that fixes it, and the one rule that
  makes it safe: it proposes, you dispose
- [**The moving parts**](the-moving-parts.md) —
  managers that read your files, datasources
  that know the versions, platforms that
  host your code, and the branch and pull
  request at the end of the pipe
- [**Installing Renovate**](installing-renovate.md) —
  the three honest ways to get it: Mend's
  hosted app, the Docker image, and the npm
  package this guide's lab used
- [**Your first dry run**](your-first-dry-run.md) —
  the local run that changes nothing:
  extraction, lookup, and reading the
  results — including a lookup that failed,
  kept in the transcript on purpose
- [**Managers — how Renovate reads a project**](managers.md) —
  how the right reader is matched to each
  file, what a dependency record holds,
  and why the lab's two-file project was
  scanned against 106 managers
- [**Datasources — where version truth comes from**](datasources.md) —
  registries as the only source of truth,
  what a lookup returns besides the number,
  and why the same lookup failed once and
  worked an hour later
- [**Configuring Renovate**](configuring-renovate.md) —
  the `renovate.json` file taken apart:
  presets, package rules, schedules,
  automerge — and the validator that checks
  your file before a run does
- [**Onboarding and the Dependency Dashboard**](onboarding-and-the-dashboard.md) —
  the first pull request Renovate ever
  sends you, and the one issue that lists
  everything it is waiting to do
- [**Branches, groups and schedules**](branches-groups-and-schedules.md) —
  how branch names are built, how grouping
  trades many small pull requests for a
  few bigger ones, and when a schedule
  keeps updates out of your working day
- [**Automerge and trust**](automerge-and-trust.md) —
  letting the robot merge what it proposed:
  the safeguards that make that sane, and
  the cases where it is not
- [**Lock files and digest pinning**](lock-files-and-digests.md) —
  the updates with no version change at
  all: lock file maintenance, and pinning
  container images by digest
- [**Running Renovate for real**](running-renovate-for-real.md) —
  the hosted app, a scheduled action, or
  your own machine: tokens, timing, and
  the security updates that jump the queue
- [**Recipes**](recipes.md) — a sensible
  starter config, a quiet config for a
  busy repository, pinning a container
  image properly, and testing a config
  change before it lands
- [**The neighbours**](the-neighbours.md) —
  Dependabot, the platform's own robot,
  compared fairly; and the other honest
  options, from doing it by hand to doing
  it never
- [**Limits**](limits.md) — what the robot
  cannot know: whether an update is safe,
  whether a changelog is honest, and what
  it does with a project nobody maintains
- [**Troubleshooting**](troubleshooting.md) —
  the warning about a missing native
  module, the lookup that hangs up, the
  preset typo the validator cannot see,
  and the pull requests that never come
- [**Glossary**](glossary.md) — the terms,
  collected in one place

## What it doesn't cover

Writing the code your project is made of —
Renovate updates the list of what your
project borrows; it does not review the
borrowing. Fixing the breakage an update
causes is likewise yours: Renovate will
faithfully open the pull request that
updates a library across a breaking change
and label it a major update, but reading
the migration guide is a human job.
Running a private registry or a corporate
artifact mirror is pointed at, not taught;
the ideas are the same and the plumbing is
a different guide. And while the lab ran
Renovate itself, it ran it in dry-run mode
against files on a disk: connecting the
robot to your real GitHub, GitLab or
Bitbucket account is covered as documented
procedure in chapters 8 and 12, not as
something this lab did.

## How to use this guide

Read chapter 1 for the model and chapter 2
before installing anything — knowing which
part knows what turns later confusion into
a named problem. Chapters 3 and 4 get you
a dry run against your own project in one
sitting, and that dry run is the safest
first step this guide knows: it changes
nothing and shows you everything Renovate
would do. Read the configuration chapter
with your dry-run output open; the two are
about each other. The lab's transcripts
live in `renovate-lab/transcripts/`,
numbered in the order the work happened,
and the little project it ran against is
in `renovate-lab/fixture/` so you can run
the same commands yourself.

---

## Part of the series

This guide is one of the [For Mere Mortals](https://paulsullivan33-dev.github.io/mere-mortals/) guides: plain-language guides to the tools programmers and makers use.

Related guides:

- [Git for Mere Mortals](https://paulsullivan33-dev.github.io/git-for-mere-mortals/)
- [GitHub Actions for Mere Mortals](https://paulsullivan33-dev.github.io/github-actions-for-mere-mortals/)
- [Docker for Mere Mortals](https://paulsullivan33-dev.github.io/docker-for-mere-mortals/)
- [Node.js for Mere Mortals](https://paulsullivan33-dev.github.io/nodejs-for-mere-mortals/)

## License

The writing in this guide is licensed under [CC BY 4.0](https://github.com/paulsullivan33-dev/renovate-for-mere-mortals/blob/main/LICENSE). Code examples and lab scripts are licensed under the [MIT License](https://github.com/paulsullivan33-dev/renovate-for-mere-mortals/blob/main/LICENSE-CODE).
