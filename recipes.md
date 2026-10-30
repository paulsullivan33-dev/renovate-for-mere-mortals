# Recipes

**The outcome of this chapter:** four configurations and
workflows you can lift whole, each with the reasoning left
in — because a recipe whose reasons you don't know is a
preset you didn't read.

## Recipe 1: a starter config that behaves

For a project adopting Renovate this week. It is the lab's
fixture config, annotated:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "schedule": ["before 6am on monday"],
  "packageRules": [
    { "matchManagers": ["npm"],
      "groupName": "npm packages" },
    { "matchUpdateTypes": ["minor", "patch"],
      "matchCurrentVersion": "!/^0/",
      "automerge": true }
  ]
}
```

Swap `npm` in the grouping rule for your own ecosystems —
one rule per ecosystem, each grouped separately, is the
shape that scales. Delete the automerge rule for your first
month (chapter 12) and add it back when chapter 10's ladder
says so. Validate before committing:

```console
$ renovate-config-validator renovate.json
INFO: Config validated successfully against 1 file(s)
```

## Recipe 2: quieting a busy repository

A large project where Renovate has become weather — always
there, never looked at. The levers, in the order to pull
them:

```json
{
  "extends": ["config:recommended"],
  "prHourlyLimit": 2,
  "prConcurrentLimit": 5,
  "schedule": ["before 6am on monday", "before 6am on thursday"],
  "packageRules": [
    { "matchUpdateTypes": ["major"],
      "schedule": ["on the first day of the month"] }
  ]
}
```

Rate limits first, because they cap the total noise;
schedules second, because they move what noise remains
into known windows; majors monthly, because they are the
updates that deserve a planned sitting. If it is still too
loud, group more aggressively before you schedule more
rarely — a monthly pile of thirty separate pull requests
is worse than a weekly group of five.

## Recipe 3: rehearsing a config change

Before changing the config of a repository that matters,
run the change past the robot locally — the same dry run
as chapter 4, used as a test bench:

```console
$ git clone <the repository> && cd <the repository>
$ # edit renovate.json the way you are considering
$ renovate-config-validator renovate.json
$ LOG_LEVEL=debug renovate --platform=local 2>&1 | \
    grep -E "flattened updates|branch=renovate/"
```

Read the branch list the way chapter 9 taught you: the
names *are* the behaviour. Groups that should exist and
don't, branches that shouldn't exist and do — both show up
here, for free, before a single real pull request is
harmed. Two limits from chapter 4 apply: presets borrowed
from other accounts cannot be resolved locally, and
platform-only behaviour (automerge, dashboard, alerts)
does not rehearse. What rehearses — extraction, lookup,
grouping, branch naming — is most of what configs get
wrong.

## Recipe 4: pinning a container image by digest

Documented-only (the lab's docker runs returned version
updates, not digest changes), but the steps are short and
the payoff is chapter 11's reproducibility:

```json
{
  "packageRules": [
    { "matchManagers": ["dockerfile"], "pinDigests": true }
  ]
}
```

Renovate then proposes a pull request that rewrites
`FROM node:18.20.4-alpine` into the same tag with its
current `@sha256:…` digest attached, and from then on
proposes digest updates whenever the tag's contents move,
and version updates exactly as before. Let it pin the
images you *deploy*; leave build-stage and throwaway
images unpinned unless you enjoy reading digest diffs —
a digest diff is the one diff in computing that tells you
*something* changed and nothing about what.

## Recipe 5: the pre-flight check for any project

The five-minute version of this whole guide, to run in any
repository before deciding anything:

```console
$ LOG_LEVEL=debug renovate --platform=local --dry-run=extract
$ LOG_LEVEL=debug renovate --platform=local
```

From the first: the dependency count and which managers
claimed which files (chapter 5). From the second: the
outdated count, the lib-years total, the branch list, and
any lookup warnings riding on individual dependencies
(chapters 4 and 6). If you do nothing else with Renovate
this year, run this pair quarterly: it is the cheapest
honest picture of a project's staleness that exists, it
changes nothing, and nobody has to install an app or hand
out a token to get it.
