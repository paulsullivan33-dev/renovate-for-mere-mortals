# Limits

**The outcome of this chapter:** you can recite what
Renovate cannot do — not as fine print, but as the
boundaries a sensible operator plans around.

**It cannot judge an update's safety.** Renovate knows a
version's number, age, and breaking *label*. It does not
know what the code does, whether the changelog is honest,
or whether your use of the library touches the changed
part. Its pull requests carry the release notes it could
find; deciding is deliberately left to you and your tests,
and chapter 10 exists because automerge spends that
judgement in advance.

**It only sees what managers can read.** A version pinned
in prose, in a script's string, in a tool Renovate has no
manager for, is invisible until a custom manager is taught
to see it (chapter 5). Extraction is broad — 106 managers
in the lab's scan — but broad is not total, and the misses
are silent: nothing in an ordinary run lists the
dependencies it *didn't* find.

**It believes registries completely.** Chapter 6's
epistemology has a flip side: a registry outage means
blindness, a poisoned or yanked release means Renovate
will faithfully propose it until the registry says
otherwise, and metadata quality — timestamps, deprecation
marks — varies by ecosystem. Minimum release ages and
schedules are sandbags, not a cure.

**Breaking labels are arithmetic, not understanding.**
`isBreaking` is computed from version numbers against the
rules of semantic versioning. Pre-1.0 software plays by
looser rules (hence the 0.x exclusion in this guide's
configs), some projects ignore the rules entirely, and
some breaking changes ship as patches by mistake or
malice. The label sorts your review queue; it does not
clear it.

**It cannot fix what an update breaks.** A failed pull
request waits for a human. Renovate will keep it rebased
and will close it if the update is superseded, but the
migration — reading the upstream guide, changing your
code — is the part of updating that was always human work.

**Its local mode is a rehearsal, not the play.** Chapter
4's dry runs are real extraction and real lookup, but the
local platform cannot create branches, cannot resolve
other accounts' presets, ignores some platform concepts,
and treats `dryRun=full` as `lookup`. Behaviour that only
exists on a platform — automerge merging, the dashboard,
vulnerability alerts — cannot be verified locally, and
this very guide had to teach those parts from
documentation. Any tool demonstration that doesn't tell
you which mode you are looking at is hiding the same seam.

**It is a wide program with a wide surface.** One binary
that understands a hundred ecosystems and holds a token
with write access to your repositories is, by construction,
a valuable thing to compromise and a lot of code to trust.
The mitigations are the boring ones from chapter 12 —
narrow token reach, few repositories per identity, config
in review — and they are mitigations, not eliminations.

**It assumes the project is alive.** Renovate will
happily keep a dead project's dependencies eternally
fresh, opening pull requests nobody will ever merge, at
the cost of CI time and notification noise. The robot has
no opinion about whether the project deserves updates.
That opinion is yours, and the off switch per repository —
closing the onboarding PR, or disabling in config — is
part of using the tool well.
