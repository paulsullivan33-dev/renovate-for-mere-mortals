# Configuring Renovate

**The outcome of this chapter:** you can read a
`renovate.json` and say what each part does, write the four
settings a new project actually needs, and check your file
with the validator — knowing exactly which mistakes the
validator cannot see.

Repository configuration lives in one file in the project.
The usual name is `renovate.json` in the repository root;
`.github/renovate.json` and a `renovate` key inside
`package.json` are among the other recognised homes, and
JSON5 and JSONC variants exist for people who want comments
in the file. Start with the plain one. The lab's file, the
one transcripts t04 and t05 ran with, is small enough to
take apart completely:

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "schedule": ["before 6am on monday"],
  "packageRules": [
    {
      "matchManagers": ["npm"],
      "groupName": "npm packages"
    },
    {
      "matchUpdateTypes": ["minor", "patch"],
      "matchCurrentVersion": "!/^0/",
      "automerge": true
    }
  ]
}
```

Every key in it is doing one of four jobs: borrowing
somebody else's settings, setting a rhythm, or matching a
set of dependencies and treating them specially. Take them
in that order.

## Borrowing: presets and `extends`

A **preset** is a published bundle of settings with a name.
`config:recommended` is Renovate's own sensible-defaults
bundle — the one nearly every project starts from. There
are presets for grouping known-related packages
(`group:monorepos` and friends), for pinning strategies,
for quiet schedules, and anyone can publish their own for
others to extend: `github>your-org/renovate-config` borrows
a file from a GitHub repository, which is the standard way
a company keeps forty projects behaving identically.

Honest warnings, both earned later in this chapter. First:
a preset is code you did not read, and it changes your
robot's behaviour exactly as surely as your own lines do —
skim any preset you adopt. Second: preset typos are the
one mistake the validator does not catch, for a structural
reason the lab demonstrated (below).

## Rhythm: `schedule`

`"schedule": ["before 6am on monday"]` says when Renovate
may *act* — create and update its branches and pull
requests. The language is human-shaped on purpose:
`before 6am on monday`, `after 10pm every weekday`, `on the
first day of the month`. Chapter 9 is about choosing a
schedule; here it is enough to know the setting exists,
applies unless a rule overrides it for particular
dependencies, and is checked by the validator, because
"whenever" — the lab's planted error — is not a schedule
and was rejected in those words.

## Matching and treating: `packageRules`

Most of Renovate's judgement lives in `packageRules`: a
list of rules, each in two halves — *match* some updates,
then *treat* them somehow. The matchers are the vocabulary
of the earlier chapters: `matchManagers` (which reader found
it), `matchDatasources`, `matchPackageNames` (and its
pattern form), `matchUpdateTypes` (patch, minor, major),
`matchCurrentVersion`, `matchDepTypes`. The treatments are
everything else in the config language: group, schedule,
enable, disable, rename the branch, add reviewers, merge
automatically.

The lab's two rules, translated:

1. **"Anything the npm manager found goes in a group named
   'npm packages'."** One pull request for the whole group
   instead of one per library. Chapter 9 shows what that
   did to the branch list — and the surprise that came
   with it.
2. **"Minor and patch updates, where the current version
   is not a 0.x version, may automerge."** Two cautions are
   baked into this rule and worth stealing. It excludes
   *major* updates entirely — breaking-flagged updates
   always wait for a human. And the odd-looking
   `"matchCurrentVersion": "!/^0/"` excludes 0.x software,
   because in the 0.x world the rules of semantic
   versioning are looser: a "minor" bump from 0.4 to 0.5
   is allowed to break things, and projects use that
   freedom. Automerging those on the strength of the word
   "minor" is how robots acquire reputations.

`automerge` itself gets a whole chapter (chapter 10)
because the switch is easy and the judgement is not. Here,
note only where the switch lives: it is a treatment like
any other, so it can be as narrow as your matching is
honest.

## Checking the file: the validator

A config typo on a real platform costs you a failed run and
a confusing issue. Renovate ships a checker so you can fail
faster, on your own machine:

```console
$ renovate-config-validator renovate.json
```

Run against the lab's good file (transcript t03):

```
INFO: Validating renovate-good.json as global config
INFO: Config validated successfully against 1 file(s)
exit code: 0
```

Run against a deliberately broken sibling — four planted
mistakes: a misspelled preset (`config:recomended`), a
string where a boolean belongs, an unparseable schedule,
and an option that does not exist — it caught three, named
them precisely, and exited 1:

```
Configuration option `automerge` should be boolean. Found: "yes please" (string)
Invalid configuration option: unknownOption
Invalid schedule: `Invalid schedule: Failed to parse "whenever"`
```

Read that error list once more. The misspelled preset is
not in it. The validator checks the file's *structure* —
option names, types, schedules — but preset names are only
resolved when Renovate runs against a platform and fetches
them, so a typo there sails through validation and fails
later, in the run. Trust the validator for what it is: a
spell-checker for the language, not for the library you
borrow from. (The same blind spot explains why the lab's
local runs leaned on presets so lightly: on the local
platform, presets from other accounts cannot be resolved
at all.)

## Global versus repository configuration

One last distinction, because confusing the two wastes
debugging hours. Some settings are **global-only**: the
platform, the token, which repositories to visit — the
runner's business, set through environment variables
(`RENOVATE_TOKEN` and kin) or a global config file, and
deliberately *ignored* if a repository's file tries to set
them, because a repository that could hand its robot new
credentials would be a security hole wearing a config file's
clothes. Everything in this chapter is repository
configuration: steerable by the project, reviewable in the
project, safe to show the project. When a setting seems to
be ignored, the first question is which layer it belongs to.

Your file is valid, your rules are written. The next thing
a real project sees is not an update at all — it is an
introduction.
