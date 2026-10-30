# The neighbours

**The outcome of this chapter:** you can choose between
Renovate and the platform's own update robot with both
eyes open, and you know the smaller tools that solve a
slice of the same problem.

## Dependabot: the obvious neighbour

**Dependabot** is GitHub's own dependency robot: it reads
a project's manifests, checks for newer versions and for
known vulnerabilities, and opens pull requests. GitHub, Inc.
bought it and built it into the platform; on GitHub it
needs no installation at all, and a small YAML file
(`.github/dependabot.yml`) sets its rhythm. Any honest
comparison starts by conceding Dependabot's home-field
advantages:

- **Zero setup on GitHub.** No app to install, no token to
  manage, no infrastructure. For a small project on GitHub
  that wants security updates and little else, this can be
  the whole argument.
- **Security alerting is native.** GitHub computes the
  vulnerability alerts from its own advisory database;
  Dependabot's security pull requests grow directly out of
  them, with no integration step in between.

Renovate's counter-arguments are the themes of this guide:

- **Breadth of ecosystems.** Dependabot covers the major
  package ecosystems well. Renovate's manager list runs
  longer — the lab counted 106 managers scanned, including
  Dockerfiles, GitHub Actions, Terraform, Helm, CI configs,
  regex-driven custom managers for the odd places, and
  several ecosystems Dependabot does not read.
- **Breadth of platforms.** Dependabot works where GitHub
  hosts. Renovate also runs on GitLab, Bitbucket, Azure
  DevOps, Gitea, and their self-hosted forms.
- **Depth of configuration.** Grouping freely across
  packages, schedules in plain language, minimum release
  ages, automerge rules by update kind and current version,
  lock file maintenance, digest pinning: Dependabot has
  some of these in simpler forms (grouping exists there
  too, and scheduled scans), Renovate has all of them,
  uniformly, in one config file that lives anywhere.
- **One robot everywhere.** A team split across GitHub and
  GitLab gets one behaviour and one config language from
  Renovate, or two robots with two dialects otherwise.

The fair summary: on GitHub, with common ecosystems and
modest needs, Dependabot is the path of least resistance
and choosing it is not a compromise. Renovate earns its
keep when you outgrow any one of: GitHub-only hosting,
per-ecosystem configuration, shallow grouping, or manual
lock file care. Running both at once on the same
repository is the one choice this chapter argues against:
two robots proposing overlapping updates is worse than
either one alone.

## The smaller neighbours

**Your package manager's own tools** — `npm outdated`,
`pip list --outdated`, `cargo update` previews and kin —
are chapter 4's extract-and-lookup done by hand: accurate,
immediate, and gone the moment you close the terminal. They
create no pull requests and no habits, but for a one-off
audit they are unbeatable, and Renovate's dry run is best
understood as their disciplined big sibling.

**Version-pinning and update CLIs** for a single ecosystem
(`npm-check-updates` is the veteran) rewrite ranges in one
file type with great polish and no interest in the rest of
your project. Fine tools; narrower promise.

**Container-image watchers** (the family that watches a
running or pinned image and tells you — or applies — when
its tag moves) solve the Docker slice of the problem from
the deployment side. They complement rather than compete:
a repository robot proposes the new digest in code review;
a watcher notices the world changed underneath a running
system. (This guide's series covers that family in its own
right elsewhere; the choice of tool is a separate reading.)

**Doing it by hand, on a calendar.** A recurring task —
"first Monday: update everything, run tests, fix fallout" —
is a legitimate strategy for exactly one project with a
disciplined owner. Its failure mode is the subject of
chapter 1: the calendar slips twice, and the next sitting
is a five-version jump under pressure. Renovate exists
because this strategy's success depends on a human's
unbroken streak of free Mondays.

**Doing nothing** deserves its sentence too: a finished,
frozen, or throwaway project may honestly not need updates.
Renovate's lib-years metric (chapter 4) at least lets you
choose that with the number in front of you.
