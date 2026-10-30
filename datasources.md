# Datasources — where version truth comes from

**The outcome of this chapter:** you can say where Renovate
gets its facts, what a lookup returns besides a number, and
why the same lookup can fail at noon and succeed at one.

A manager's record ends with a pointer: *ask the npm
datasource*, *ask the docker datasource*. The **datasource**
is Renovate's half of a conversation with a registry — the
npm registry, Docker Hub, PyPI, Maven Central, crates.io,
GitHub releases, an operating system's package archive, and
dozens of others. Registries are the only places that know
what versions of something exist. Renovate believes them,
and nothing else: not blog posts, not a project's README,
not the version a search engine remembers.

That is a deliberate, and mostly excellent, epistemology.
It has one sharp edge this guide will not hide: if a
registry is wrong, slow, or unreachable, Renovate is
exactly that wrong, slow, or blind — for that dependency,
in that run. The rest of the chapter is about what a
successful lookup buys you and what a failed one costs.

## What a lookup returns

"Newer version exists" is the least of it. From the lab's
npm lookups (transcript t02), each candidate update
arrived carrying:

- **The new version and its kind** — `major`, `minor`,
  `patch`, computed against your current version, with
  majors flagged `isBreaking`. This classification, not the
  raw number, is what rules and schedules later match on.
- **Release timestamps, both ends** — your version's age
  (lodash 4.17.20: 2,269 days) and the candidate's (33 days
  old, for chalk 6.0.1). Renovate can be told to hold very
  fresh releases back for a few days (`minimumReleaseAge`)
  on the theory that a version pulled by its publisher
  within a week of release is one you are glad to have
  missed. Documented-only here — the lab observed the
  timestamps the policy would act on.
- **A digest** — for npm, the integrity hash of the exact
  tarball (`sha512-…`). This is the fingerprint chapter 11's
  pinning is built from.
- **Provenance** — the registry it came from, the package's
  source repository and homepage, so the pull request can
  link you to the project's home and its release notes.

It also caches aggressively. The lab's first lookup phase
took 10.7 seconds, mostly network; a later run the same
hour reported **zero HTTP requests** and a 1.1-second
lookup, answering entirely from its cache of registry
responses. When a lookup result puzzles you, "the cache
remembers an older answer" belongs on your list of
suspects, next to the remedies in the troubleshooting
chapter.

## The failure, and the recovery

Chapter 4 kept the lab's first Docker Hub attempt verbatim:
the registry's sign-in service (`auth.docker.io`) reset
the connection mid-handshake — `ECONNRESET`, "socket hang
up" — three retries, then Renovate logged `docker registry
connection failure`, attached the warning `node: socket
hang up` to that one dependency, and finished the run
without it.

An hour later, same command, same fixture, one config file
added: the lookup succeeded (transcript t04). The `node`
dependency had **two** updates queued where nothing was
wrong with the versions themselves: `18.20.4-alpine` could
become `18.20.8-alpine` (patch) or `26.13.0-alpine` (major).
Note the shape of that answer, because it teaches two
things at once:

- Renovate offers updates along more than one line. Staying
  on Node 18's maintenance line and jumping to the current
  line are separate proposals on separate branches
  (`renovate/node-18.x`, `renovate/node-26.x`) — you are not
  forced to ride the newest major to get your patches.
- The `-alpine` suffix survived untouched. The log calls it
  `currentCompatibility`, and the docker datasource treats
  the variant as part of the deal: updates stay within the
  flavour of image you chose unless you say otherwise.

Why did it fail once and work once? The honest answer is
that public registries are shared infrastructure with bad
minutes, and sandboxed networks add their own ideas about
which hosts may be spoken to. The lab never established
which of those it was, and the guide won't pretend
otherwise — the operational lesson stands either way:
**Renovate degrades per-dependency and reports the wound.**
A missing update with a warning attached is the system
working as designed, not a mystery to solve at midnight.

## Private registries and credentials

Everything above used public registries, which need no
credentials to read. If your project pulls from a private
registry — a company npm mirror, a private container
registry — the datasource needs a **host rule**: Renovate's
name for a matched block of "for this host, use this
credential". Host rules are global configuration, supplied
by whoever runs the robot through environment or a config
file, and they are the right place for secrets: tokens stay
out of the repository's own `renovate.json`, which is
reviewed in public like any other file. This guide's lab
used no credentials at all — that was a rule of the lab,
not a gap in it — so private-registry setup is
documented-only here. The shape to remember: match on the
host, keep the secret in the runner's environment, and
never write a token into a file that gets committed.

With the facts gathered, what remains is judgement: which
of these updates to propose, in what company, at what
hour, and with what promise about merging. That judgement
is written down in one file, and it is next.
