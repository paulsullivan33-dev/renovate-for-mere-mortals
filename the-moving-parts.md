# The moving parts

**The outcome of this chapter:** given any Renovate
behaviour — an update found, an update missed, an update
grouped with another — you can name which part produced it.

Renovate's work is a pipeline of four stages, each handled
by a different kind of part. The stages have names in the
project's own logs, and this guide uses the logs a lot, so
it is worth meeting the cast first.

## Managers read your files

A **manager** knows one kind of file. The `npm` manager
reads `package.json`. The `dockerfile` manager reads
Dockerfiles. Others read Python requirements, Go module
files, Maven POMs, GitHub Actions workflows, Terraform
files, Helm charts, and more than a hundred further kinds.

A manager's whole job is to answer one question: *which
dependencies does this file declare, and at what versions?*
Its output is a list of **dependency records**. Here is one
from the lab's run, slightly trimmed (transcript t01):

```json
{
  "depName": "chalk",
  "currentValue": "4.1.2",
  "datasource": "npm",
  "depType": "dependencies",
  "packageName": "chalk"
}
```

Five facts, and each matters. The name and current version
are obvious. `depType` says where in the file the dependency
sat — ordinary `dependencies`, as opposed to a development
dependency or a build tool. And `datasource` is the
manager handing the record on: *to know whether this is out
of date, ask the npm datasource.* Managers never look
anything up themselves. They read; asking is another part's
job.

## Datasources know the versions

A **datasource** knows one kind of registry: where versions
of things live. The `npm` datasource asks the npm registry.
The `docker` datasource asks a container registry such as
Docker Hub. There are datasources for PyPI, Maven Central,
GitHub releases, GitLab releases, crates.io, the NuGet
gallery, operating-system package archives, and dozens more.

Given a dependency record, the datasource returns the list
of versions the registry knows, with extras the robot needs
to behave politely: when each version was released, whether
a version is deprecated, whether the project's current
version still exists at all, and a fingerprint (**digest**)
of the exact bits, where the registry provides one.

Datasources are the only part of Renovate that tells it
facts about the outside world. That has a consequence this
guide's lab demonstrated twice: if a datasource cannot
reach its registry, everything downstream of it stops —
for that dependency only — and the rest of the run carries
on. Chapter 6 is built around the transcripts.

## Platforms host your code

A **platform** is where the project lives: GitHub, GitLab,
Bitbucket, Azure DevOps, Gitea, and a few others. The
platform part reads the repository's files, and — in a real
run — creates the branches and pull requests, reads their
status, and merges the ones you have told it to merge.

There is one special platform, and it is the one this
guide's lab used: `local`. The local platform reads files
from a directory on disk and creates nothing. It exists
for **dry runs** — rehearsals — and it changes the meaning
of the run in ways chapter 4 spells out. The lab could
watch extraction and lookup in full, for real, because
those stages do not need a platform's write access. The
stages past lookup — branches, pull requests, merges — are
the ones the local platform cannot perform, and in this
guide they are labelled documented-only wherever they
appear, because the lab did not perform them.

## Configuration steers all of it

Over the whole pipeline sits **configuration**: settings
that say which updates to propose, how to group them, when
to propose them, and which may merge themselves. There are
two layers:

- **Global configuration** belongs to whoever runs the
  robot: which platform, which repositories, the access
  token. It lives outside the projects.
- **Repository configuration** belongs to the project: a
  file named `renovate.json` (or one of several equivalent
  places) in the repository itself. This is the file
  chapter 7 takes apart — presets it extends, `packageRules`
  that match some dependencies and treat them specially,
  schedules, automerge switches.

Renovate also keeps one living document per repository,
the **Dependency Dashboard**: an issue on the platform that
lists what it intends to do, what it is holding back, and
what went wrong. Chapter 8 covers it.

## The pipeline, whole

Put together, one run is:

1. **Init** — read global config, connect to the platform,
   read the repository's config file, notice whether the
   repository has been **onboarded** (chapter 8).
2. **Extract** — every manager scans the files it knows;
   dependency records come out.
3. **Lookup** — every datasource is asked about its
   records; for each dependency, Renovate works out which
   newer versions are acceptable updates and what kind each
   is: patch, minor, major, or a digest-only change.
4. **Update** — acceptable updates are grouped according to
   the config, turned into branches, and offered as pull
   requests, each with the release notes the datasource
   could supply.

The lab's logs print these stage names and how long each
took: in one run, `{"init": 788, "extract": 216, "lookup":
10660, "update": 0}` milliseconds. Lookup took fifty times
longer than extraction, because lookup is the stage that
talks to the internet. Remember that shape; it explains
most of Renovate's timing, its caching, and its rate
limits.

Next: getting the robot onto a machine — yours, or Mend's.
