# Session state — resume here next time

## Where we are — M1 shipped

- **M0 scaffold + M1 DoIP/UDS/DTC work**: committed in local scratch
  repo at `/tmp/bench-mercedes`, shipped as
  `bench-mercedes-m1.bundle` (full) and
  `bench-mercedes-m0-to-m1.bundle` (incremental) in this directory.
- **Plan**: `docs/mercedes-diag-plan.md` on this branch.
- **Hard out-of-scope** (do not drift): anti-theft/VIN-lock write on
  head units; redistribution of Mercedes proprietary files; SCN.

## What M1 added

- DoIP framing + TCP adapter with routing activation, response-
  pending handling, AliveCheck auto-reply
- DoIP UDP discovery helper
- UDS: ReadDTC-by-status-mask, ClearDTC, EcuReset, SecurityAccess
  seed/key scaffolding, RoutineControl, TesterPresent
- UDS 3-byte DTC parser with 7-char code formatting
- Avalonia UI: IP/port/tester/ECU fields, Connect/Disconnect,
  Read/Clear DTCs in DataGrid, VIN display
- Tests: DoIP header encode/decode/invalid-inverse, DTC parsing

## Next session — first actions

1. User imports `bench-mercedes-m1.bundle` into their
   `rohite1983/Bench-mercedes` repo on their own machine.
2. User runs `dotnet restore && dotnet build && dotnet test` and
   reports any errors — no `dotnet` in sandbox so I couldn't verify.
3. If clean, user plugs ENET cable into a bench car and tries
   Connect → Read DTCs. Any runtime failures get diagnosed next
   session.
4. Then **M2**: PCAN + ISO-TP for pre-2015 vehicles.

## Open items

- Confirm the sandbox scope has been (or will be) expanded to
  include `rohite1983/Bench-mercedes`. Until then, keep using the
  bundle workflow.
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
