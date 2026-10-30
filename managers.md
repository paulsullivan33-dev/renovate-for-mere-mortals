# Managers — how Renovate reads a project

**The outcome of this chapter:** when Renovate misses a
dependency you know a project has — or invents one you
don't recognise — you know which stage to interrogate, and
the three questions that settle it.

Chapter 2 introduced the manager's bargain: *give me a file
I understand and I will tell you the dependencies in it.*
This chapter is about how that bargain is struck, because
extraction is the foundation the whole pipeline stands on.
A lookup can only update what a manager found; a pull
request can only change what a manager can safely rewrite.

## Matching files to managers

A manager announces itself with file patterns. Renovate's
debug log prints them as the extraction stage starts, and
the lab's log is a small education by itself. Against a
project containing exactly two dependency files, the scan
tried, among others:

```
Using file pattern: /(^|/)package\.json$/ for manager npm
Using file pattern: /(^|/)([Dd]ocker|[Cc]ontainer)file$/ for manager dockerfile
Using file pattern: /(^|/)go\.mod$/ for manager gomod
Using file pattern: /(^|/)pom\.xml$/ for manager maven
Using file pattern: /(^|/)Cargo\.toml$/ for manager cargo
Using file pattern: /(^|/).tool-versions$/ for manager asdf
```

… and so on, for **106 distinct managers** in the lab's
run, from Ant to WordPress plugins. This is why Renovate
can walk into an arbitrary repository and be useful in one
pass: "what counts as a dependency file here?" is a
question it answers with a very long list of patterns,
not with configuration. If your project declares a
dependency in a file with a conventional name, some
manager almost certainly has a pattern for it.

The practical consequence cuts both ways. A dependency
declared in a *unconventional* place — a version number
typed into a shell script, a wiki page, a comment — is
invisible to extraction, and no amount of lookup will save
it. Renovate has an escape hatch for that case: **custom
managers**, where you supply your own pattern (a regular
expression over file contents) and name the datasource to
ask about matches. That is a documented-only corner in
this guide — the lab's fixture used only built-in managers —
but it is the right tool when a real project pins a version
in a README example or a home-grown manifest, and the
configuration chapter points at it again.

## What a dependency record holds

Chapter 2 showed a trimmed record. The whole of it, for
the fixture's base image (transcript t01):

```json
{
  "depName": "node",
  "packageName": "node",
  "currentValue": "18.20.4-alpine",
  "replaceString": "node:18.20.4-alpine",
  "autoReplaceStringTemplate": "{{depName}}{{#if newValue}}:{{newValue}}{{/if}}{{#if newDigest}}@{{newDigest}}{{/if}}",
  "datasource": "docker",
  "depType": "final"
}
```

Two fields here earn their keep later. `replaceString` is
the exact text in the file Renovate will later rewrite —
`node:18.20.4-alpine` in the Dockerfile — and the template
beside it is how a replacement is rebuilt when a new value
(or a new digest) arrives. Extraction is therefore not
just reading: it is also noting *how to write*. Managers
that cannot safely rewrite a spot tend not to claim it.

`depType` is subtler than it looks. In the fixture's
package file it reads `dependencies`. Had the same library
been listed under `devDependencies`, the record would say
so — and your configuration can treat the two differently
(chapter 7's rules match on dep type, among other things).
The Dockerfile record's `final` says this image is the
stage the build ships, not an intermediate builder stage;
a multi-stage Dockerfile yields one record per stage.

## Lock files ride along

Managers also notice **lock files** — the files (`package-lock.json`,
`yarn.lock`, `poetry.lock`, `Gemfile.lock`, and kin) that
record not what a project *allows* but what it *actually
installed*, down to every transitive dependency — the
dependencies of the dependencies. When the lab added a
lock file to the fixture (transcript t05), the npm
manager's record grew two lines:

```json
"managerData": { "npmLock": "package-lock.json", ... },
"lockFiles": ["package-lock.json"]
```

That changes what an update *does*: with a lock file in
play, Renovate's pull request must regenerate the lock file
as well as bump the number, and a whole extra class of
update — lock file maintenance, chapter 11 — exists just
to refresh the pins nobody declared. If a real project's
pull requests from Renovate ever surprise you by touching
a huge lock file over a one-line version change, this is
why, and it is correct behaviour: the two files must agree,
or the next install will not reproduce.

## Interrogating extraction

When extraction misbehaves, three questions settle nearly
every case, all answerable from the extract dry run:

1. **Did any manager claim the file?** Search the debug
   log for the file name. No claim means no pattern matched
   — the file is named or placed unconventionally, or the
   ecosystem needs a custom manager.
2. **Did the claim produce the record you expected?** The
   "Extracted dependencies" block in the log is the ground
   truth of what the rest of the run will work with. A file
   can be claimed and still yield nothing if its contents
   don't parse the way the manager expects — an unusual
   version format is the usual culprit.
3. **Is the version in the record the version in the file?**
   Ranges complicate this honestly: a `package.json` saying
   `"lodash": "^4.17.20"` declares a range, and the record
   then carries the range as `currentValue` with the locked
   or installed version tracked separately. Renovate handles
   ranges by proposing range updates or pinning, steerable
   by config (`rangeStrategy`), but the first step is
   noticing that the record reflects a *range*, not a pin.

Extraction is also, not coincidentally, the fastest stage
of the run — 216 milliseconds in the lab's timed run —
because it reads only local files. Everything slow,
network-bound, and occasionally unreliable lives in the
next chapter's territory: asking the registries.
