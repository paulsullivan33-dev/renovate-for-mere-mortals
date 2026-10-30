# Branches, groups and schedules

**The outcome of this chapter:** given a project's branch
names from a Renovate run, you can explain how each was
composed, choose between one-pull-request-per-update and
grouping, and set a schedule that keeps the robot's work
out of your way — without silencing it.

## Branch names are sentences

Renovate works on branches named by a template, and the
default template produces names that parse like sentences:

```
renovate/chalk-6.x        — dependency chalk, heading for the 6.x line
renovate/lodash-4.x       — dependency lodash, staying in the 4.x line
renovate/node-18.x        — the base image, patch line
renovate/node-26.x        — the base image, new major line
```

Those four are all real, from the lab's transcripts. The
pattern is `renovate/<dependency>-<target line>`, and it
encodes the decision chapter 6 described: patches on your
current line and the jump to a new major line are separate
branches, separately reviewable, separately mergeable. You
can merge the safe one today and argue about the major all
month. The branch name told you which was which before you
opened anything.

Groups rewrite the name. A group — created by `groupName`
in a package rule, or by a grouping preset — collects
several updates onto one branch named for the group, and
the pull request carries all of them together.

## The grouping experiment

The lab's config grouped every npm dependency under the
name "npm packages" (chapter 7). The un-grouped run had
returned two branches; the grouped run returned
(transcript t04):

```
DEBUG: 4 flattened updates found: node, node, chalk, lodash
DEBUG: Returning 4 branch(es)
      branch=renovate/node-18.x
      branch=renovate/node-26.x
      branch=renovate/major-npm-packages
      branch=renovate/npm-packages
```

The two Docker updates were untouched, not being npm. But
the npm pair did **not** become one branch. It became two,
and the split is the lesson: Renovate's default
`separateMajorMinor` is true, and it outranks grouping.
Major updates are grouped separately from non-major ones
even inside a named group — hence `major-npm-packages` for
chalk's jump to 6.0.1, and plain `npm-packages` for lodash's
minor step. The reasoning is the same as chapter 7's rule:
a major update deserves its own review, its own test run,
and its own merge decision; folding it into a bundle of
safe updates would smuggle the risk past the reviewer. If
you truly want them together, `separateMajorMinor` can be
switched off per rule — know that you are switching off a
safety rail, not a cosmetic default.

## What grouping buys and costs

Grouping is a trade, and both sides are real.

**One pull request instead of nine** means one CI run
instead of nine, one review sitting instead of nine, and
one merge. For a stable project with a pile of minor
updates, grouping by ecosystem (all npm, all GitHub
Actions, all Docker) or by a monorepo's known-related
packages — the `group:monorepos` presets exist because some
libraries are versioned in lockstep and *cannot* sensibly
be updated separately — turns a chore into a glance.

**The cost is blast radius and diagnosis.** A grouped pull
request with one bad update fails as a whole; finding which
of the nine broke the build is your afternoon. And a group
moves at the speed of its slowest member: one dependency
whose update conflicts can hold eight good ones hostage.

The working compromise most projects land on, and the one
this guide recommends starting from: group the routine
(non-major, same ecosystem), never group majors, and keep
security-driven updates ungrouped and unscheduled (chapter
12), because those are the ones whose whole value is speed.

## Schedules: when, not whether

A schedule in Renovate never cancels an update; it decides
when branches and pull requests are *created and refreshed*.
Off-schedule, pending updates wait — visible on the
Dependency Dashboard, chapter 8 — and a human can always
summon one early from there.

The lab's file used `"schedule": ["before 6am on monday"]`:
one quiet weekly window. Renovate's own presets suggest
the common shapes — weekday evenings, weekend mornings,
the first week of the month for majors. Three honest
considerations choose between them:

- **Your CI capacity is the real limit.** Every open or
  refreshed pull request spends a test run. Renovate also
  has explicit rate limits (`prHourlyLimit`,
  `prConcurrentLimit`) for the same reason; a schedule is
  the coarse version of the same kindness.
- **Freshness has a security price.** A monthly schedule
  means a patched library can sit unproposed for weeks
  unless vulnerability updates are exempted — Renovate can
  run those "at any time" regardless of the general
  schedule, and chapter 12 argues it should.
- **Predictability beats cleverness.** A team that knows
  "Renovate morning is Monday" builds a habit around it.
  Scattered arrivals get ignored, and ignored pull requests
  rot: Renovate will keep the branch rebased and the
  request current, but a robot cannot make a review happen.

Set the rhythm, then, like a standing meeting: fixed,
known, and with an exceptions list. The next question is
whether any of these pull requests should be allowed to
finish the job themselves.
