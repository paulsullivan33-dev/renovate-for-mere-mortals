# Glossary

The terms this guide uses, collected in one place. Where a
term has a meaning outside Renovate, this is the meaning
the guide relies on.

**Automerge** — a setting under which Renovate merges a
pull request itself once its required checks pass. Scoped
by package rules; argued about, properly, in chapter 10.

**Branch** — the unit of proposed change. Renovate creates
one branch per update (or per group of updates), named by
template — `renovate/chalk-6.x` — and the pull request is
opened from it.

**Config file** — the repository's own Renovate settings,
usually `renovate.json` in the project root. Steers
grouping, scheduling, rules and automerge. Chapter 7 takes
one apart.

**Datasource** — the part of Renovate that knows one kind
of registry and can list the versions it holds, with
timestamps, deprecation marks and digests. Managers read;
datasources answer.

**Dependency** — something a project borrows and names a
version of: a library, a tool, a container image, an action.

**Dependency Dashboard** — the standing issue Renovate
maintains on a repository listing open, pending, and
errored updates, with checkboxes to summon pending ones
early. Chapter 8.

**Digest** — a cryptographic fingerprint of one exact
artefact (`sha256-…` for images, `sha512-…` for npm
tarballs). Tags name lines; digests name things.

**Dry run** — a Renovate run that reads and looks up but
writes nothing. The local platform's default level is
`lookup`; `extract` stops after reading. Chapter 4.

**Group** — several updates collected onto one branch and
one pull request under a `groupName`, trading review
granularity for quiet. Majors group separately from
non-majors by default. Chapter 9.

**Host rule** — global configuration matching a host and
supplying its credential, for private registries. Secrets
live here and in the runner's environment — never in a
repository file.

**Lib-years** — Renovate's running total of staleness: the
summed ages of the gaps between current and latest
versions, as if one library were that many years behind.

**Local platform** — the special platform that works on a
directory on disk and creates nothing. The rehearsal stage
for extraction, lookup, grouping and config testing.

**Lock file** — the file recording exactly what was
installed, including transitive dependencies and integrity
hashes: `package-lock.json`, `yarn.lock`, `Gemfile.lock`
and kin.

**Lock file maintenance** — a scheduled Renovate update
that regenerates only the lock file, refreshing every pin
the manifest's ranges already allow. Chapter 11.

**Manager** — the part of Renovate that knows one kind of
dependency file and can both read the dependencies in it
and rewrite a version in place. 106 of them were scanned
in this guide's lab. Chapter 5.

**Onboarding** — the state of having a merged config file.
Renovate's first pull request on a new repository proposes
exactly that file, and it does nothing else until the file
lands. Chapter 8.

**Package rule** — one entry in `packageRules`: a set of
matchers (manager, datasource, package name, update type,
current version) and a treatment (group, schedule,
automerge, disable…) applied to whatever matches.

**Platform** — where the code lives: GitHub, GitLab,
Bitbucket, Azure DevOps, Gitea — or `local`, the folder on
a disk.

**Preset** — a named, shareable bundle of settings,
borrowed with `extends`: Renovate's own `config:recommended`,
grouping bundles, an organisation's shared config.

**Registry** — where versions of things are published and
fetched: the npm registry, Docker Hub, PyPI, Maven Central
and their private counterparts. Renovate's only source of
truth about the outside world.

**Schedule** — the human-shaped timetable ("before 6am on
monday") saying when Renovate may create and refresh
branches and pull requests. It delays; it never cancels.

**Transitive dependency** — a dependency of a dependency:
borrowed by your project without being named by it, and
therefore living only in the lock file.

**Update type** — Renovate's classification of an update
by version arithmetic: `patch`, `minor`, `major`, plus
`digest` and `lockFileMaintenance` for the changes with no
version bump. Majors are flagged `isBreaking`.

**Vulnerability alert** — a platform's notice (on GitHub,
a Dependabot alert) that a version in use has a known
security flaw. Renovate can read them on the platform and
let those updates jump schedules and groups. Chapter 12.
