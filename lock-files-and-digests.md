# Lock files and digest pinning

**The outcome of this chapter:** you can explain the two
kinds of update that change no version number in any file
you wrote — and why both exist.

Every update so far edited a number you could point at:
4.1.2 becomes 6.0.1. This chapter is about the updates
underneath those, the ones a version-number-only tool
would miss entirely.

## Lock files: the truth below the ranges

Chapter 5 introduced lock files: where the manifest says
what a project *allows*, the lock file records what it
*actually got* — including every **transitive** dependency,
the libraries your libraries borrow, which no file of yours
names at all. A manifest can sit unchanged for a year while
its lock file quietly pins a hundred packages, some of
which have had security fixes for months.

Renovate therefore treats a lock file as a living thing in
two ways:

1. **Any ordinary update regenerates it.** Bump `lodash` in
   `package.json` and the pull request also rewrites
   `package-lock.json`, because the two must agree or the
   next install is a lottery.
2. **Lock file maintenance updates the lock file alone.**
   On a schedule, Renovate opens a pull request whose only
   change is the lock file, regenerated against the ranges
   the manifest already allows — pulling every transitive
   pin up to the newest version the project already said it
   accepts.

The lab ran the second kind. With a `package-lock.json`
committed (created by `npm install --package-lock-only`,
holding the fixture's exact old versions) and one config
line added — `"lockFileMaintenance": { "enabled": true }`
— the next run reported five branches instead of four
(transcript t05). The new one:

```
branch=renovate/lock-file-maintenance-npm-packages
```

Two details in that name reward attention. It exists at
all only because the option was switched on — lock file
maintenance is an opt-in with a schedule of its own
(weekly in the stock presets) precisely because it produces
pull requests on a timer whether or not anything "changed".
And it inherited the fixture's group name, because lock
file maintenance passes through `packageRules` like
everything else — the `npm packages` grouping rule matched
it without being asked twice. Your rules are a system, and
new kinds of update join it whether you planned for them
or not; the dashboard (chapter 8) is where you would watch
that happen.

One honest boundary of the lab's evidence: a local dry run
proves the branch would be created and what the run
intended. The regenerated lock file itself is written in
the platform phase the lab could not perform, so the exact
contents of such a pull request are documented-only here.

## Digests: pinning the thing itself

A version tag names a line of releases. A **digest** names
one exact artefact — a fingerprint (for npm tarballs,
`sha512-…`; for container images, `sha256-…`) such that if
a single byte changes, the fingerprint does too. Chapter
6's lookup results carried digests for the npm updates:
Renovate tracks them because two different safety stories
depend on them.

**Story one: the moving tag.** Container tags are not
promises. `node:18.20.4-alpine` today and next month are
*supposed* to be identical, but a registry lets a publisher
re-point a tag, and rebuilds and mistakes happen. Writing
the digest into the file —

```dockerfile
FROM node:18.20.4-alpine@sha256:1e9f2b3c… (illustrative)
```

— converts "whatever that tag means now" into "exactly
these bits". With `pinDigests` enabled, Renovate proposes
digest pins for images that lack them, and thereafter
proposes **digest updates**: same tag, new digest, flagged
as update type `digest`, when the tag's contents move.
The lab did not run this — its Dockerfile used a bare tag,
and its successful docker lookup returned version updates
(t04), not digest changes — so digest behaviour in this
guide is documented-only. The shape above is the whole
idea; tooling specifics are in the project's docker
datasource documentation.

**Story two: the lock file parallel.** A digest pin and a
lock file are the same instinct at different scales: stop
trusting names, record fingerprints. `package-lock.json`
stores integrity hashes for every package it pins, which
is why a maintenance pull request that "only regenerates
the lock file" is really proposing a new set of
fingerprints for a hundred packages — worth the same
glance you would give any other change of substance.

## Which to use, when

- **Applications you deploy** benefit from both: lock files
  maintained, images digest-pinned. Reproducibility is the
  product.
- **Libraries you publish** usually skip digest pinning
  (your consumers resolve their own) but still gain from
  lock file maintenance in development.
- **Either costs review attention.** A digest bump is a
  one-line diff with a long shadow; a lock file regeneration
  is a huge diff with, usually, no shadow at all. Renovate
  can't tell those apart for you — but it can schedule the
  noisy one weekly and the quiet ones as they come, which
  is most of the answer.

That completes the mechanics. The last operational chapter
puts them all in a running system.
