# Automerge and trust

**The outcome of this chapter:** you can state the
conditions under which letting Renovate merge its own pull
requests is sane engineering rather than laziness — and
the conditions under which it is neither.

Every chapter so far kept the robot at arm's length: it
proposes, a human merges. **Automerge** removes the human
from the merge click for the updates a rule describes.
Chapter 7 showed the switch as one line in a package rule.
This chapter is about earning the right to flip it.

## What automerge actually does

An automerging pull request is opened like any other. The
difference comes after: Renovate watches the pull request's
required checks — the platform's branch protection decides
what "required" means — and, when they pass, merges it
itself. Variants exist (merge the branch without a pull
request at all; ask the platform to merge once its own
checks pass), but the shape is constant: **automerge is
conditional on green.** A failing update never automerges;
it sits in the pull request queue like any other failure,
waiting for a human to look.

That one property is load-bearing for everything below. If
it is ever not true of your setup — if a pull request can
merge while its checks are red or absent — you do not have
automerge, you have auto-approval, and this chapter does
not apply to you until that is fixed.

## The honest case for it

Some updates carry almost no information for a reviewer.
A patch release of a mature library, in a project whose
test suite genuinely exercises that library, merged forty
times a year by a human who has stopped reading the diff —
that human is not a safety mechanism, they are a delay
mechanism wearing a safety mechanism's hat. Automerge,
narrowly scoped, converts their rubber stamp into a rule
that never gets tired, never merges on a red build because
it was Friday, and leaves an audit trail in the config
file. The lab's own fixture rule (chapter 7) was shaped
like the recommendation: minor and patch updates only,
0.x software excluded, everything else waits.

## The honest case against, case by case

- **Your tests are the whole argument.** Automerge trusts
  CI completely. If the suite does not cover the dependency
  being updated — and coverage is thinnest exactly where
  code is oldest — a green build means "nothing we tested
  broke", which can be a much weaker claim than it feels.
  The robot cannot know the difference. You can.
- **Majors are labelled, not understood.** `isBreaking`
  comes from the version arithmetic, and chapter 2's
  classification is honest — but a minor release can still
  change behaviour the changelog understates, and a patch
  can still be a compromised release. The 0.x exclusion and
  the major exclusion exist because the label is a
  heuristic, and heuristics get a vote, not a veto-proof
  mandate.
- **Freshness is a risk factor of its own.** A release
  published an hour ago has survived an hour of scrutiny
  by the world. Waiting periods (`minimumReleaseAge`) are
  the standard counterweight and pair naturally with
  automerge: let the ecosystem's early adopters go first.
- **One bad merge teaches the wrong lesson.** When an
  automerged update breaks production — it will, someday —
  the failure worth fixing is the test gap that let it
  through. Teams that respond by switching automerge off
  entirely have traded a real, bounded risk for the
  unbounded risk of chapter 1: everything quietly going
  stale again.

## A trust ladder

If you adopt automerge, climb rather than leap:

1. **Watch.** A few weeks of ordinary Renovate, merging by
   hand, while you note which updates you never even
   hesitated over. That set is your automerge candidate
   list — measured, not imagined.
2. **Patch only, mature only.** Automerge patch updates
   for dependencies past 1.0 with a track record, in
   projects with suites you trust. Keep minors human.
3. **Add minors where the evidence supports it.** Per
   package, per ecosystem — rules are cheap, and a rule
   per trusted dependency is finer control than a global
   switch.
4. **Never majors, never 0.x, never the platform itself.**
   Breaking-flagged updates, pre-1.0 software, and updates
   to the pipeline that runs the pipeline stay human
   decisions. This rung is not a stage on the way somewhere
   else; it is the top.

And keep the kill switch in reach: automerge lives in the
repository's config file, so revoking it is one reviewed
commit — which is as it should be, for a robot whose whole
design premise is that the humans remain able to say stop.
