# Session state — resume here next time

## Where we are — M1.1 shipped (.NET 10 upgrade)

- **M0 scaffold + M1 DoIP/UDS/DTC work + M1.1 .NET 10 upgrade**:
  committed in local scratch repo at `/tmp/bench-mercedes`, shipped
  as `bench-mercedes-m1.1.bundle` (full) and
  `bench-mercedes-m1-to-m1.1.bundle` (incremental) in this directory.
- User successfully pushed M0+M1 commits to
  `https://github.com/rohite1983/Bench-mercedes` (private) from his
  Mac earlier this session via `gh auth login` + `git push`.
- **Plan**: `docs/mercedes-diag-plan.md` on this branch.
- **Hard out-of-scope** (do not drift): anti-theft/VIN-lock write on
  head units; redistribution of Mercedes proprietary files; SCN.

## What M1.1 added

- `Directory.Build.props`: `<TargetFramework>` bumped net8.0 → net10.0,
  `<LangVersion>` 12 → latest.
- Dropped `global.json` (was pinning SDK 8.0.100, which forced an
  older install; user has 10.0.202).
- Package bumps:
  - `Microsoft.Extensions.Hosting` 8.0.1 → 10.0.0
  - `Microsoft.Extensions.Logging.Abstractions` 8.0.1 → 10.0.0
  - `Serilog.Extensions.Hosting` 8.0.0 → 9.0.0
  - Test SDK bumps for net10 compat (17.11 → 17.12, xunit 2.9.2 →
    2.9.3, runner 2.8.2 → 3.0.0, FluentAssertions 6.12.1 → 6.12.2).
- Avalonia 11.2.1 and CommunityToolkit.Mvvm 8.3.2 untouched — both
  multi-target net10 fine.

## Next session — first actions

1. User runs:
   ```
   cd ~/path/to/Bench-mercedes
   git pull /path/to/artifacts/bench-mercedes-m1-to-m1.1.bundle main
   git push origin main
   dotnet restore
   dotnet build
   dotnet test
   dotnet run --project src/MercedesDiag.App
   ```
   Report any build/test errors — no `dotnet` in sandbox so I can't
   verify locally.
2. If clean, user plugs ENET cable into a bench car and tries
   Connect → Read DTCs. Any runtime failures get diagnosed next
   session.
3. Then **M2**: PCAN + ISO-TP for pre-2015 vehicles.

## Open items

- Confirm the sandbox scope has been (or will be) expanded to
  include `rohite1983/Bench-mercedes`. Until then, keep using the
  bundle workflow. User opted for "Expand sandbox scope" earlier
  this session; awaiting the admin change on the Claude Code side.
- Sanity-check the default vehicle IP (currently 169.254.0.1) against
  what the user's actual ENET cable gets assigned — may need to
  switch to DoIP UDP discovery as the primary connect flow.
- UI language: still English-only. Revisit in M2 or M3.

## Known caveats in M1 that may surface

- DoIP response-pending handling loops forever inside the timeout
  window — fine for normal ECUs, but a broken ECU could hang the
  connection. Future: bound the number of pending-loops.
- No flow control yet for ECUs that send multi-segment responses
  needing keep-alive — not likely for standard UDS over DoIP but
  worth checking against a real target.
- `ReadDtcs` unconditionally enters the extended session; some
  ECUs reject this. May need to fall back to default session.
- No security access handshake wired into the UI yet — just APIs
  on UdsClient. UI flow for seed/key lands in M4.

## Sandbox signing note

Commits in the scratch repo at `/tmp/bench-mercedes` are unsigned
because the sandbox signing server rejects writes from that path
(`missing source`). All three commits (M0, M1, M1.1) follow the
same pattern. Signatures re-applied when the user pushes from his
Mac, so this is cosmetic.
