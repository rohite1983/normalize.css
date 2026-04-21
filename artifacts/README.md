# Session artifacts

Unrelated to `normalize.css` itself. These files were produced in a
Claude Code session whose harness pinned the working branch to this
repository; they're the deliverables of that session's work on a
separate project and live here only because this was the one repo
the sandbox could push to.

## Current state: M1 committed

- `bench-mercedes-m1.bundle` — **latest, full history (M0 + M1).**
  Contains two commits: the M0 scaffold and the M1 DoIP+UDS+DTC work.
  This is the file to use if you haven't imported the project yet.
- `bench-mercedes-m0-to-m1.bundle` — incremental, M1 commit only.
  Use this if you already imported M0 and want to fetch just the new
  commit on top.
- `bench-mercedes-m1.tar.gz` — plain tarball of the M1 tree, in case
  the bundle path is awkward.

### Fresh import (empty Bench-mercedes repo)

```bash
git clone artifacts/bench-mercedes-m1.bundle Bench-mercedes
cd Bench-mercedes
git remote add origin https://github.com/rohite1983/Bench-mercedes.git
git push -u origin main
```

### Already imported M0, adding M1

```bash
cd Bench-mercedes
git pull /path/to/artifacts/bench-mercedes-m0-to-m1.bundle main
git push origin main
```

## Historical

- `bench-mercedes-m0.bundle` / `bench-mercedes-m0.tar.gz` — the
  original M0 scaffold, kept for reference. Prefer the M1 bundles
  above.

## After import: verify on your machine

```bash
dotnet restore
dotnet build
dotnet test
dotnet run --project src/MercedesDiag.App
```

You should see the Avalonia window open with connection fields,
an ECU dropdown, and empty DTC grid. If `dotnet build` fails, paste
the errors into the next session and I'll fix them.
