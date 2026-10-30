# Installing Renovate

**The outcome of this chapter:** Renovate is runnable on a
machine you control — or you have knowingly chosen the
hosted app instead — and you can prove it with one command
that prints a version and changes nothing.

There are three honest ways to get Renovate, and they suit
different readers.

**The hosted app.** Mend, the company behind Renovate, runs
it as a service and offers it in the GitHub Marketplace (and
for GitLab). You install the app on your account, choose
which repositories it may see, and Mend's machines do the
running. Nothing to install, nothing to keep updated. This
is the route most people should take first, and chapter 12
describes what happens after you click install. This guide's
lab did not take it — a lab cannot click on a marketplace
listing — so everything about the app's behaviour in these
pages is documented-only, and says so.

**The Docker image.** Mend publishes an official image,
`renovate/renovate`. One `docker run`, given a configuration
and a token, does one pass over your repositories and exits.
This is the standard self-hosted route and the one the
project's own documentation assumes; chapter 12 shows the
shape of it. The lab's sandbox cannot run containers, so
this route, too, is documented-only here.

**The npm package.** Renovate is a Node.js program and is
published to the npm registry as the package `renovate`.
If you have Node.js, you can install the genuine article
and run exactly the binary the other routes run. This is
the route the lab took — the only one its machine allowed —
and everything in chapters 4 to 11 was done with it.

## The lab's install

The lab installed Renovate into a folder of its own rather
than system-wide, which is the tidy way to try any npm
program you may later delete:

```console
$ mkdir renovate-labwork && cd renovate-labwork
$ npm_config_cache=$PWD/npm-cache \
    npm install --prefix tools --no-audit --no-fund renovate
```

Two details are worth knowing. The `--prefix` keeps the
program and its several hundred support packages inside
`tools/`, where deleting one folder removes the lot. And
the install is not small — the lab's folder came to about
350 MB. Renovate carries support for a hundred-plus
ecosystems in one program, and that breadth has a size.

Proof it works (transcript t00):

```console
$ node --version
v24.22.1
$ npm --version
10.9.4
$ renovate --version
44.198.2
```

`renovate --version` printing a number is the whole check:
the program starts, its own machinery loads, and it exits
having touched nothing. If you installed the npm package
globally instead (`npm install --global renovate`), the
same command works from anywhere; the lab's copy lives at
`tools/node_modules/.bin/renovate` under its prefix.

## One warning you will meet

Every run of the lab's npm-installed Renovate opened with
this warning:

```
WARN: RE2 not usable, falling back to RegExp
```

RE2 is a small native library — a regular-expression engine
that is guaranteed not to run catastrophically slowly on
hostile input — which Renovate uses to check patterns
safely. The npm install did not build the native piece on
the lab's machine, so Renovate noted the fact and used
Node's built-in engine instead. Nothing in the lab's
results depended on it, and the official Docker image
ships RE2 built, so most readers will never see this. It
is in the guide because you might, and because a warning
that appears on every run deserves one paragraph that says
*this one is known, and what it costs you*: pattern checks
are a little less armour-plated. If it troubles you, the
Docker route makes it go away.

## Keeping it current

Whichever route you chose, Renovate itself releases
constantly — the version the lab ran was current the week
it was installed and will not be by the time you read this.
The hosted app updates itself. The Docker route updates by
pulling a newer image — and yes, Renovate can be configured
to propose that update to its own deployment, which is
either elegant or vertiginous depending on your temperament.
The npm route updates the way this chapter installed it:
run the install again. A version printed by
`renovate --version` before and after is your proof.

Now the reason you came: pointing it at a project.
