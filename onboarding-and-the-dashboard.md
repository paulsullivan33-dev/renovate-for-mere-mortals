# Onboarding and the Dependency Dashboard

**The outcome of this chapter:** you know what the first
pull request Renovate sends a project actually contains,
why nothing happens until you merge it, and where the one
standing summary of the robot's intentions lives.

*Everything in this chapter describes behaviour on a real
hosting platform. The lab had no platform account, so this
is documented-only material, assembled from the project's
documentation — except where it says the lab saw the shape
of it in a dry run, which it did, twice.*

## The onboarding pull request

Install Renovate on a repository (chapter 12) and its first
act is not an update. It is a pull request, conventionally
titled **Configure Renovate**, proposing to add one file:
the `renovate.json` the project will run with. Until that
pull request is merged, the repository is **not onboarded**,
and Renovate does essentially nothing else to it.

The lab met this machinery from the side. Its fixture had
no config file at first, and the local run's log says so in
platform language:

```
DEBUG: Repo is not onboarded
INFO: DRY-RUN: Would commit files to onboarding branch
DEBUG: onboarding config
      "config": {"$schema": "https://docs.renovatebot.com/renovate-schema.json"}
```

That `Would commit` line is the dry run being literal: on a
real platform, a branch would be created and the pull
request opened; here, the intent is logged and nothing is
written. The proposed file, in a plain repository with no
organisation defaults to borrow, is exactly what the log
shows — the `$schema` line and nothing else, meaning "run
with the defaults". Where an organisation publishes a shared
default config, the onboarding file extends that instead.

Why the ceremony? Because it is the moment the robot's
existence enters the project's own review process. The
onboarding pull request's text explains what Renovate will
do, what it found in the repository, and what happens next.
Editing the proposed config *inside* that pull request is
the intended way to start a project with chapter 7's
settings already in place; merging it as-is is equally
intended, and you can change the file any time after, like
any other file. Closing the pull request without merging
is also a complete sentence: Renovate reads it as "not
here", and stays away unless invited again.

Once the lab added a `renovate.json` to the fixture, the
log flipped to `DEBUG: Repo is onboarded` and stayed there.
Onboarding is a property of the repository, not of the
run — checked at the start of every run, satisfied by the
mere presence of a config file.

## The Dependency Dashboard

A robot that opens pull requests one at a time can be hard
to hold in your head: which updates are coming, which are
being held back by a schedule or a rate limit, which failed
and why. Renovate's answer is the **Dependency Dashboard**:
a single issue on the repository, created and continuously
rewritten by Renovate, that lists its state of mind.

Turned on with one setting (`"dependencyDashboard": true`),
the dashboard's sections read like a status board:

- updates already open as pull requests;
- updates waiting — for their schedule, for a rate limit on
  how many pull requests Renovate may open per hour, or
  because a previous pull request for the same thing was
  closed by a human and Renovate is politely not re-asking;
- updates that errored, with the error — a failed registry
  lookup of the kind chapter 4 kept is dashboard material;
- and, usefully, checkboxes: tick the box beside a pending
  update and Renovate creates that pull request on its
  next run, schedule or no schedule. The dashboard is not
  just a report; it is the one sanctioned way to hurry the
  robot up.

The lab cannot show you one — dashboards live on platforms,
and the lab's platform is a folder. What it can confirm
from its own logs is the need: a single twelve-second local
run produced an onboarding decision, three dependency
records, two found updates, one failed lookup and a small
pile of warnings. Spread over forty repositories and a
month of runs, that is exactly the volume of fact a
dashboard exists to hold in one place.

Newer hosting also offers a web-based Dependency Dashboard
view in Mend's developer platform; the issue-based original
remains the one that works everywhere Renovate does,
including self-hosted, and is the one this guide means by
the name.

Onboarded, dashboarded, configured. What remains is the
craft of the updates themselves: how many branches, of
what company, arriving when.
