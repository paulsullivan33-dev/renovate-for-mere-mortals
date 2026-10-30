# Troubleshooting

**The outcome of this chapter:** the six failures a new
Renovate operator actually meets, each with the check that
identifies it and the fix that closes it. Every failure
below was met by this guide's lab unless it says otherwise.

## The warning about RE2 on every run

```
WARN: RE2 not usable, falling back to RegExp
```

**What it means:** the optional native regular-expression
library was not available to this installation — the usual
cause is an npm install that did not build native modules.
Renovate continues with Node's built-in engine, and warns
that pattern validation may be less strict.

**What to do:** nothing, for most purposes; the lab's
results were unaffected. If the warning bothers you or
your config leans heavily on patterns, use the official
Docker image, which ships RE2 built. Do not go hunting for
it in your *config* — nothing there causes or cures it.

## A dependency gets a warning instead of an update

The lab's first Docker Hub lookup (chapter 4) ended like
this, verbatim:

```
WARN: docker registry connection failure
      "message": "socket hang up"
```

and the dependency record carried `"warnings": [{"topic":
"Lookup Error", "message": "node: socket hang up"}]`.

**The check:** find the dependency's own record in the
debug output and read its warnings — a dependency that
*could not be looked up* says so there. A dependency that
genuinely has no newer version has empty warnings and an
empty updates list, which is a different and healthier
answer.

**The fix:** usually time. Registries and networks have
bad minutes; the lab's identical retry later succeeded.
If it persists, test the registry directly (can this
machine reach the registry host at all? is a proxy in
between selective about hosts?) before suspecting your
config — datasources fail per-dependency precisely so this
stays a small problem.

## The validator passes but the run complains about a preset

Chapter 7's planted typo, `config:recomended`, passed
validation cleanly and would only fail when a real run
tried to resolve it. **Why:** the validator checks the
file's structure — option names, types, schedules — while
preset *names* are resolved against a platform during a
run. **The fix:** spell preset names exactly as their
documentation gives them, and treat a green validator as
"well-formed", never as "will run". When a run does fail
on a preset, the error names the preset it could not find;
check for transposed letters first. It is always
transposed letters.

## No pull requests appear

Work the pipeline backwards, in chapter 2's order:

1. **Onboarded?** No merged onboarding PR (or config file)
   means Renovate politely does nothing (chapter 8). The
   log line is `Repo is not onboarded`.
2. **Extracted?** Run the extract dry run (recipe 5). If
   the dependency isn't in the extraction output, no
   downstream stage can invent it — chapter 5's three
   questions apply.
3. **Looked up?** In the full dry run, does the dependency
   have updates, or warnings (above), or neither? Neither
   can also mean an update exists but rules excluded it:
   an `enabled: false` match, a schedule holding it, a
   version already at latest.
4. **Held, not lost?** On a platform, check the Dependency
   Dashboard before concluding anything: rate limits
   (`prHourlyLimit` exhausted for this hour), a schedule
   window in the future, and "a human closed this PR
   before" all park updates *visibly* — the dashboard is
   where parked things report themselves.

## The updates stopped after working for months

The usual suspects, in the order the lab would check:
the bot's token expired or lost reach (the runner's logs
will say so, in authentication language); the repository
was renamed or the bot's installation was suspended; a
config change introduced an error — run the validator,
then a local dry run, and compare; or the registry changed
something (a deprecated datasource endpoint, a moved
package), which shows up as per-dependency lookup warnings
rather than silence. Total silence from a configured,
onboarded repository almost always means the *runner*
stopped — a deleted cron job, an expired action, a lapsed
app installation — before it means the robot did.

## Automerge merged something it shouldn't have

First, the fact to establish: **which rule matched.** The
pull request's branch and the config's rule list answer
it; automerge never freelances. Then ask the harder
question from chapter 10: did the rule match as intended
but the *checks* fail to cover the change — a test gap —
or did the rule itself overreach (a `matchUpdateTypes`
covering more than you pictured, a 0.x dependency slipping
past a missing `matchCurrentVersion` guard)? Fix the layer
that actually failed; if it was the test gap, say so in
the post-mortem, because switching automerge off entirely
treats the symptom by restoring chapter 1's disease.
