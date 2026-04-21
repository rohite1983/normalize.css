# Session state — resume here tomorrow

## Where we are

- **M0 scaffold**: committed and shipped as `bench-mercedes-m0.bundle`
  in this `artifacts/` directory. Not yet imported into the
  `rohite1983/Bench-mercedes` repo on the user's GitHub.
- **Plan**: `docs/mercedes-diag-plan.md` on this branch.
- **Hard out-of-scope** (do not drift on these): anti-theft/VIN-lock
  write on Audio20/COMAND/NTG; redistribution of Mercedes proprietary
  files (SMR-D, CBF, .aed); SCN online coding.

## Next session — first actions

1. Ask the user: did you import the bundle into `Bench-mercedes` and
   run `dotnet restore && dotnet build && dotnet test`? Any errors?
2. If clean, move to **M1**: implement DoIP transport
   (`MercedesDiag.Transport/Doip/DoipChannel.cs`,
   `MercedesDiag.Hal/Doip/DoipAdapter.cs`) and the minimum UDS
   service set (`0x10`, `0x22`, `0x27`, `0x19`, `0x14`, `0x11`,
   `0x31`). Wire ECU-list + connect + read-DTCs into the Avalonia
   UI.
3. Need to know before M1 starts: which ENET cable / hardware the
   user has on the bench.

## Sandbox / delivery reminder

- This environment's git auth is scoped to `rohite1983/normalize.css`
  only. Cannot push to `Bench-mercedes` directly. Keep using this
  branch as the sandbox-crossing channel, or ask the user to expand
  scope.
- `dotnet` is not installed in the sandbox — cannot verify builds
  here. Changes must be reviewed by the user running `dotnet build`
  on their machine.

## Open questions still unresolved

- ENET / J2534 / PCAN hardware on the bench.
- UI language(s): English only, or multi-language from day one
  (Arabic + French were hinted).
- Whether user wants the Bench-mercedes repo scope added to this
  sandbox, or prefers the bundle-handoff workflow.
