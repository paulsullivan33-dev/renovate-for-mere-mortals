# Running Renovate for real

**The outcome of this chapter:** you can choose between the
three ways Renovate runs against real repositories, give
the robot an identity with the right reach and no more,
put it on a clock, and make security updates jump every
queue you have built.

*This chapter is documented-only throughout. The lab ran
Renovate in local dry-run mode only — no platform accounts,
no tokens, nothing with the power to write to a real
repository. That was deliberate, and this chapter is
written so you can go further than the lab did, with your
eyes open.*

## The three deployments

**Mend's hosted app.** Install it from the GitHub
Marketplace (Mend offers the equivalent for GitLab),
approve the repositories it may touch, and merge the
onboarding pull request (chapter 8). Mend's infrastructure
runs the robot on its own schedule. Choose this unless you
have a reason not to: it is the only route with nothing to
operate.

**A scheduled action.** On GitHub, the project's official
action (`renovatebot/github-action`) runs Renovate inside
your own CI, on a cron schedule you write, with a token you
supply. You operate a little more — the workflow file, the
token's renewal — and in exchange the runs happen inside
your own fences.

**Fully self-hosted.** The Docker image (or the npm package
of chapter 3) on any machine or scheduler you like —
a cron job, a systemd timer, a Kubernetes CronJob — pointed
at GitHub, GitLab, Bitbucket, Azure DevOps, Gitea, or your
own instance of one of those. This is the route for private
platforms and strict networks, and the one where every
operational fact in this chapter is yours to manage.

## The robot's identity, and its token

However it runs, Renovate acts as a **bot account** (or a
platform app installation) with an access **token**. Two
rules govern that credential, and both are the kind of
obvious that bears printing:

- **It is a key to every repository it can reach.** Give it
  the narrowest reach that works: only the repositories it
  serves, only the permissions it needs (read the code,
  write branches and pull requests, read checks). Prefer the
  platform's fine-grained or app-based credentials over a
  personal token tied to a human who might leave.
- **It never lives in a repository file.** Tokens are
  global configuration — environment variables
  (`RENOVATE_TOKEN`, and per-registry credentials in host
  rules, chapter 6) or the hosted app's own vault. A token
  found in a committed `renovate.json` is a token to rotate,
  today.

Onboarding behaviour follows from the identity: the app
route's repositories are the ones you approved at install;
a self-hosted runner discovers repositories by the token's
reach (`autodiscover`) or by an explicit list in its global
config. Start with an explicit, short list. You can always
lengthen it from a position of knowledge.

## Putting it on a clock

A self-hosted Renovate does one pass and exits; the
schedule that runs *it* is separate from the schedules
*inside* the repositories' configs (chapter 9), and
confusing the two is a classic first-week bug: the runner's
cron decides when the robot wakes, the repository schedules
decide what it may do while awake. Hourly waking with
per-repository schedules doing the shaping is a sound
default. The hosted app makes the first half Mend's
problem, which is much of its appeal.

## Security updates jump the queue

Everything so far treated all updates alike, but one class
is not like the others: updates that fix a known
vulnerability in a version you are running. On GitHub,
Renovate can read the repository's **vulnerability alerts**
(the same Dependabot alert data the platform computes) and
respond to them specially: a pull request raised promptly,
labelled with the alert, even when the repository's
schedule says "Mondays" and its grouping says "everything
together". The `vulnerabilityAlerts` settings control the
etiquette — labels, assignees, and whether the schedule is
bypassed. This guide's recommendation: let them bypass.
A schedule is a courtesy to reviewers; a published exploit
is not interested in courtesy.

The lab's one brush with this machinery is honest in its
smallness: every local run logged `No vulnerability alerts
found` — not because the fixture was safe (its lodash was
six years old) but because the local platform has no alert
feed to read. Vulnerability handling is a platform feature,
and claiming otherwise would be exactly the kind of
overreach `criteria.md` exists to prevent. Treat this
section as a map of what to switch on, drawn from the
project's documentation, and verify it on your platform
with one deliberately old dependency before you rely on it.

## A sane first month

1. Install on one low-stakes repository. Merge the
   onboarding PR with a starter config (chapter 13 has one).
2. Live with the pull requests for two weeks. Tune grouping
   and schedule where the noise actually is.
3. Turn on the Dependency Dashboard; make it the team's
   one place to look.
4. Only then consider automerge (chapter 10's ladder), and
   only after the token's reach has been reviewed once,
   deliberately, by a human who is not you.
