# What Renovate is

**The outcome of this chapter:** you can say what problem
Renovate solves, what it actually does about it, and the one
rule that makes handing it the keys acceptable.

Almost no program is written from scratch. A small web app
borrows a framework, a date library, a colour library, a
database driver — each written by strangers, each improving
(or being patched for security holes) on its own schedule.
Alongside the code sit the machines it runs on: a container
image with a language runtime inside it, released fresh
every few weeks. All of that borrowed material is listed,
somewhere in the project, as names and version numbers:

```json
"dependencies": {
  "chalk": "4.1.2",
  "lodash": "4.17.20"
}
```

```
FROM node:18.20.4-alpine
```

Those two fragments are this guide's lab fixture — a tiny
invented project, a family recipe box app. They are also a
perfect picture of the problem. The versions in them were
current once. `lodash` 4.17.20 was released in August 2020.
Left alone, the list does not drift toward safety; it drifts
toward age. Every project you have ever met has this list,
in one file format or another, quietly getting older.

The manual cure is known to be miserable: remember to check
a dozen registries, read a dozen changelogs, edit a dozen
files, run the tests a dozen times. So it happens late, in
a panic, usually when a security advisory forces it — and a
jump of five versions at once, made under pressure, is the
riskiest upgrade of all.

## The robot

**Renovate** is a program that takes over the noticing and
the editing, and leaves the deciding to you. Given access
to a project hosted on GitHub, GitLab, Bitbucket or another
supported platform, it works roughly like this:

1. It reads every file in the project that can name a
   dependency — package files, lock files, Dockerfiles,
   build scripts, infrastructure definitions, and many
   more — and builds the list of what the project borrows,
   with the version currently in use.
2. For each item on the list, it asks that item's home
   registry — the npm registry, Docker Hub, PyPI, Maven
   Central, and so on — what versions exist now.
3. For each item that is behind, it prepares an update:
   a separate branch with the version number changed, and
   a pull request proposing that change, with the release
   notes it could find attached.
4. It keeps its own proposals tidy: when a newer version
   appears, it refreshes its open pull requests; when you
   merge or close one, it adjusts the rest; when an update
   it proposed is no longer needed, it closes it.

Renovate is open-source software (AGPL-3.0-only), built by
the company Mend and a large community, and it has been
doing this job for years across most of the ecosystems a
mere mortal is likely to touch — this guide's lab watched
it scan a two-file project against the file patterns of
**106** different dependency managers.

## The rule that makes it safe

A robot that edits your project sounds alarming, so here is
the property the whole guide rests on, stated before any
procedure depends on it:

**Renovate proposes. It does not dispose.** Every update
arrives as a pull request — the same artefact a human
contributor would open — and nothing lands in your code
until the pull request merges. By default, merging is your
click. If you later choose to let Renovate merge some
classes of update itself (chapter 10), that is a rule you
wrote, in a config file you control, applying to updates
you described — and you can delete the rule.

Two habits follow from that rule, and this guide teaches
both:

- **Start with a dry run.** Renovate has a mode that reads
  and looks up but writes nothing — not even a branch. The
  lab used it for every run in this guide, and chapter 4
  shows you the same thing against your own project. You
  can watch the robot think before it ever touches anything.
- **Configure in the open.** Everything Renovate does is
  steered by one configuration file kept *inside* the
  project, reviewed like any other code. Chapter 7 takes
  that file apart.

## What it is not

Renovate is not a security scanner: it will tell you a
newer version exists, and on some platforms it can see the
platform's own vulnerability alerts and prioritise those
updates (chapter 12), but it does not audit your code for
weaknesses. It is not a guarantee that an update is safe —
a pull request that passes your tests can still change
behaviour your tests do not cover, which is why chapter 10
exists. And it is not the only robot: GitHub's own
Dependabot does a similar job for projects hosted there,
and chapter 14 compares the two fairly, because choosing
between them is a real decision and "the one built into
your platform" is sometimes the right answer.

That is the model. Chapter 2 names the parts inside it —
because when Renovate behaves oddly, the oddity always
belongs to one of the parts, and naming it is most of the
diagnosis.
