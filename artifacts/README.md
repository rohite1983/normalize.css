# Session artifacts

Unrelated to `normalize.css` itself. These files were produced in a
Claude Code session whose harness pinned the working branch to this
repository; they're the deliverables of that session's work on a
separate project and live here only because this was the one repo
the sandbox could push to.

## `bench-mercedes-m0.bundle`

A git bundle containing the M0 scaffold commit for the **Bench
Mercedes** project (a personal/workshop Mercedes diagnostic tool).
To import it into an existing empty repo on your machine:

```bash
# on your own machine, after `gh repo clone rohite1983/Bench-mercedes`
# (or `git clone https://github.com/rohite1983/Bench-mercedes.git`)
cd Bench-mercedes
git pull /path/to/bench-mercedes-m0.bundle main
git push -u origin main
```

If the repo is brand new with no `main` yet:

```bash
git clone /path/to/bench-mercedes-m0.bundle Bench-mercedes
cd Bench-mercedes
git remote add origin https://github.com/rohite1983/Bench-mercedes.git
git push -u origin main
```

## `bench-mercedes-m0.tar.gz`

Same files as a plain tarball, in case the bundle path is awkward.
Extract it, then `git init` / copy the tree into your repo manually.
