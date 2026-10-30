# Writing criteria for this guide

Every "for Mere Mortals" guide is written to the same nine-point
standard, so a reader who trusts one can trust the rest. Here it
is, with the verification notes for this particular guide below.

1. **Fix it first.** Reader outcomes lead, not action items: each
   chapter opens by naming the outcome it delivers, shows a check
   the reader can run to confirm they got it, and a reasonable
   reader finishes able to state what was done, what it means,
   and what to try next.
2. **Plain language.** Short sentences, everyday vocabulary,
   technical terms defined on first use. No slang, and no
   idioms that only make sense in one country — the reader may
   be translating as they go.
3. **No dead ends.** A guide never tells the reader to do
   something it cannot show them how to verify. Every claim a
   reader relies on — an install, a command, an output — is
   something this guide demonstrates or clearly labels
   documented-only below.
4. **Live-verified where it matters.** Procedures are run for
   real in the guide's lab wherever the build machine allows it,
   and the transcripts kept in the repository are the evidence.
   What could not be run is listed honestly below, never
   presented as if it had been.
5. **General audiences.** Nothing in this guide — examples,
   fixtures, names, addresses — is modelled on any particular
   reader or author's real life, network, or organisations.
   The lab's project is an invented recipe box app with two
   stock libraries and a stock base image.
6. **Standalone.** A reader needs this guide, its lab, and
   nothing else. Where another guide would genuinely help, it
   is pointed to once, by name, and the chapter still completes
   without it.
7. **Honest about limits.** Chapter 15 names what the tool
   cannot do: judge an update's safety, see dependencies no
   manager can read, know more than the registries tell it,
   understand a breaking change beyond the arithmetic of its
   version numbers, or fix what an update breaks.
8. **The ethics rule is in the text, early.** Renovate acts on
   other people's code with a credential that can write to it,
   so chapter 1 states the governing rule before any procedure
   depends on it — Renovate proposes, humans dispose — and
   chapter 12 states the credential rules with it: narrowest
   reach that works, and never a token in a committed file.
9. **Builds clean.** The site builds with MkDocs in strict mode
   with no warnings, and a fresh reader following the chapters
   in order reaches the stated outcome of each.

## Verification notes for this guide

The build machine is a shared sandbox (about 8 GB of memory
in total, shared with other work while the lab ran; timings
are therefore approximate, and the guide says so where it
quotes them). Node.js **v24.22.1** and npm **10.9.4** were
already installed. Renovate **44.198.2** — the current
release on the day of the lab — was installed from the npm
registry into a local prefix inside the lab's own working
folder (about 350 MB with its support packages), not
system-wide; the guide's transcripts quote that copy's own
output. Its licence, AGPL-3.0-only, was read from the
installed package itself. All outbound traffic from the lab
passes through an HTTP proxy, which is part of the story
below: some registries answered through it, one at first
did not, and the guide reports both.

The fixture is the guide's own and deliberately generic: an
invented "recipe box" project (`renovate-lab/fixture/`)
whose `package.json` pins two real, old library versions —
chalk 4.1.2 and lodash 4.17.20 — and whose Dockerfile stands
on a real, old base image, `node:18.20.4-alpine`. Versions
quoted as "latest" in the transcripts (chalk 6.0.1, lodash
4.18.1, Node 26.13.0) are what the registries answered on
2026-10-30, the day of the lab; they will drift, and the
guide teaches the reading of such answers rather than the
answers themselves.

Run for real, with transcripts as evidence:

- Install and version: `renovate --version` → 44.198.2,
  alongside the Node and npm versions (t00). Every run of
  this copy opened with the warning `RE2 not usable, falling
  back to RegExp` — the npm install did not build the
  optional native module — and that warning is carried in
  the guide (chapters 3 and 16), not edited out.
- Extraction (`--dry-run=extract`): 2 package files, 3
  dependencies — dockerfile manager: `node` 18.20.4-alpine;
  npm manager: chalk 4.1.2, lodash 4.17.20 — with the full
  dependency records printed by the tool (t01). The same
  log shows the extraction scan trying file patterns for
  106 distinct managers against this two-file project.
- Lookup (local platform default, `dryRun=lookup`): the npm
  datasource answered through the proxy — chalk 4.1.2 →
  6.0.1 (type major, flagged breaking, branch
  `renovate/chalk-6.x`, with release timestamps and a
  sha512 digest in the record) and lodash 4.17.20 → 4.18.1
  (type minor, branch `renovate/lodash-4.x`); summary
  `outdated: 2, total: 3` and 10.79 npm lib-years (t02).
- A lookup failure, kept verbatim on purpose: the docker
  datasource's first attempt died at Docker Hub's sign-in
  service — `GET https://auth.docker.io/token?… =
  (code=ECONNRESET…)`, `WARN: docker registry connection
  failure`, `"message": "socket hang up"` — and the `node`
  dependency finished that run with a Lookup Error warning
  and no updates, while the npm results completed normally
  (t02).
- The validator (`renovate-config-validator`): a sound
  config passes (`Config validated successfully`, exit 0);
  a config with four planted faults exits 1 with three
  precisely named errors (a string for `automerge`, an
  unknown option, an unparseable schedule) — and the fourth
  fault, a misspelled preset (`config:recomended`), is
  demonstrably *not* caught, which chapters 7 and 16 both
  teach from this transcript (t03).
- A configured run: with the validated config committed
  into the fixture, the log shows the repository onboarded,
  the config read and applied, and the branch plan changed
  — grouping produced `renovate/npm-packages` and,
  because `separateMajorMinor` defaults to true,
  `renovate/major-npm-packages`; and the Docker Hub lookup
  that had failed in t02 succeeded, returning node →
  18.20.8-alpine (patch) and 26.13.0-alpine (major) with
  the `-alpine` compatibility suffix preserved (t04).
- Lock file behaviour: `npm install --package-lock-only`
  created a real `package-lock.json`; extraction then
  reported `npmLock`/`lockFiles`, and with
  `lockFileMaintenance` enabled the run planned a fifth
  branch, `renovate/lock-file-maintenance-npm-packages`
  (t05). A repeat run also demonstrated the cache: 0 HTTP
  requests and a 1.1-second lookup against the cold run's
  10.7 seconds.

Documented only — described from the project's documentation,
not run, and the text does not claim otherwise:

- Everything past the lookup stage on a real platform:
  branch and pull request creation, PR bodies and release
  notes as rendered, rebasing and auto-closing behaviour.
  The lab's platform was `local`, which creates nothing.
- The hosted Mend app (Marketplace installation, its
  onboarding pull request as delivered, the web Dependency
  Dashboard) and the GitHub Action deployment. The
  onboarding *machinery* was observed in the dry run's logs
  ("Repo is not onboarded", "DRY-RUN: Would commit files to
  onboarding branch", the default onboarding config), and
  chapter 8 is explicit about which half of the story that
  is.
- The Dependency Dashboard issue as rendered on a platform,
  including its checkboxes.
- Automerge actually merging: chapter 10 reasons from the
  documented preconditions (required checks green) and the
  config semantics exercised in the lab; no merge was or
  could be performed locally.
- Vulnerability alerts: every local run logged `No
  vulnerability alerts found`, because the local platform
  has no alert feed. Chapter 12 labels the feature
  documented-only and advises verifying it on the reader's
  own platform.
- Digest pinning and digest updates for container images
  (`pinDigests`): the lab's Dockerfile carried a bare tag
  and its successful docker lookup returned version updates,
  not digest changes. Chapter 11 and recipe 4 present the
  setting from documentation, labelled as such.
- Private registries and host rules with credentials: the
  lab used no tokens anywhere, by rule. Platforms other
  than the local one, custom (regex) managers, and preset
  resolution from other accounts — the last of these was in
  fact observed failing locally ("The platform you're using
  (local) does not support local presets") and is quoted as
  observed, not as documentation.
- The neighbour tools of chapter 14 (Dependabot and the
  rest) — characterised from their documentation, not run.

Fixtures are this guide's own and deliberately generic: an
invented recipe box project, stock libraries at old
versions, and a stock base image. No real project, person,
organisation or credential appears anywhere in the lab, and
no token of any kind was created, used, or needed.
