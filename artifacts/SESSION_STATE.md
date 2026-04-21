# Session state — resume here next time

## Where we are — M2.0 + M1.6 shipped

- **Scratch repo**: `/tmp/bench-mercedes` (rebuilt each sandbox session
  from the latest cumulative bundle — commits are unsigned there, see
  signing note at the bottom).
- **Delivery branch on this repo**: `claude/clarify-project-requirements-Qz5IO`.
  Each increment lands here as a `bench-mercedes-mX.Y-to-mX.Z.bundle`
  in `/artifacts/`.
- **User's real repo**: `https://github.com/rohite1983/Bench-mercedes`
  (private) on his Mac at `/Users/mohammedouchrif/Bench-mercedes`.
- **Plan file**: `docs/mercedes-diag-plan.md` on this branch.
- **Hard out-of-scope** (do not drift): anti-theft/VIN-lock write on
  head units; redistribution of Mercedes proprietary files; SCN.

## Milestone timeline shipped so far

| Commit  | Label  | What it did                                               |
|---------|--------|-----------------------------------------------------------|
| 31a4bc6 | M0     | Avalonia solution scaffold                                |
| 7d6da3b | M1     | DoIP transport + UDS services + DTC UI                    |
| 43edd0b | M1.1   | Upgrade to .NET 10                                        |
| 6577c0b | M1.2   | Fix NuGet build failures on .NET 10                       |
| 571b5b1 | M1.3   | Satisfy .NET 10 IDE style rules in build                  |
| 2954c03 | M1.4   | Drop unused MercedesDiag.Transport using                  |
| 8b5b090 | M1.5   | Fix Avalonia 11.2 XAML (no ColumnSpacing/RowSpacing)      |
| 1024256 | M2.0   | ISO-TP (ISO 15765-2) channel implementation               |
| a1d6bca | M1.6   | Expand ECU catalog + custom target address                |

## What M2.0 added

- `src/MercedesDiag.Transport/IsoTp/`:
  - `IsoTpChannel.cs` — full ISO 15765-2 single/multi-frame send &
    receive with BlockSize / STmin flow-control, background pump,
    SemaphoreSlim request gate.
  - `IsoTpOptions.cs` — padding byte (default `0xCC` for Mercedes),
    block size, STmin, flow-control timeout, frame length.
  - `IsoTpPci.cs` — PCI type and flow-status enums + STmin decoder.
  - `IsoTpException.cs`.
- `tests/MercedesDiag.Uds.Tests/Fakes/FakeCanAdapter.cs` —
  in-memory ICanAdapter with separate tx/rx `Channel<CanFrame>`s.
- `tests/MercedesDiag.Uds.Tests/IsoTpChannelTests.cs` — 6 unit tests
  covering SF round-trip, FF+CF reassembly, multi-frame send with
  FC, timeout, sequence mismatch, empty-payload rejection.
- Expected total after pull: **21 tests passing** (15 existing + 6 new).

## What M1.6 added

- `MercedesEcuCatalog.Common` expanded from 10 entries to **27**:
  added DDM/PDM/RDM-R/RDM-L, HVAC, SRS, KG, EHPS, HU, COMAND, A20,
  STH, RSL, SAM-F, SAM-R, IC, PTS, TPM, ISM, EAS, ESP, VGS, TCM-9G,
  DTR, ME, EZS, CGW.
- `MainWindowViewModel.cs`: new `UseCustomTargetAddress` bool and
  `TargetAddressHex` string. When toggle is on, `ConnectAsync`
  parses the hex field and uses it instead of `SelectedEcu`.
  Hex parser accepts `0x`-prefix or bare hex, case-insensitive.
- `MainWindow.axaml`: CheckBox "Custom target address" + TextBox
  below the ECU ComboBox; TextBox enabled when toggle is on, and
  the ComboBox is disabled when the toggle is on so the UI makes
  it obvious which address will be used.

## Next session — first actions

User on his Mac:
```
cd /Users/mohammedouchrif/Bench-mercedes
git pull ~/normalize.css/artifacts/bench-mercedes-m2.0-to-m1.6.bundle main
git push origin main
dotnet restore
dotnet build
dotnet test
dotnet run --project src/MercedesDiag.App
```

Expected after pull:
- 27 ECUs in the dropdown (previously 10).
- A "Custom target address" checkbox + hex textbox below the
  dropdown. Ticking it lets him type any logical address in hex.
- 21 tests passing.

## Next milestone — M2.1 (KWP2000 client)

- `src/MercedesDiag.Kwp/KwpClient.cs` — ISO 14230 service mapping
  for pre-2012 Mercedes over CAN (KWP-on-CAN) — many services are
  UDS-compatible but `0x1A` ReadEcuIdentification and a handful of
  others need a KWP-specific path.
- Unit tests fed from the existing FakeCanAdapter.
- No UI wiring yet; that lands with M2.3 when the adapter selector
  is added.

## Then — M2.2 (PCAN backend) and M2.3 (J2534)

- `src/MercedesDiag.Hal/Pcan/PcanAdapter.cs` implementing
  `ICanAdapter` via P/Invoke to `PCANBasic.dll` (Windows) /
  `libpcan` (Linux). Platform guards so macOS builds but shows
  "PCAN driver not available on macOS".
- `src/MercedesDiag.Hal/J2534/J2534Adapter.cs` — P/Invoke to a
  vendor J2534 DLL. Windows-only in v1; other platforms compile
  a stub that throws `PlatformNotSupportedException`.
- UI: an adapter-selector dropdown (ENET / PCAN / J2534 / C3-C4)
  with per-adapter settings panels.

## Open items

- Sandbox scope for `rohite1983/Bench-mercedes` still not granted,
  so the bundle workflow continues.
- Vehicle IP default `169.254.0.1` still unverified against the
  user's actual ENET cable — if it doesn't come up, we'll switch
  Connect to kick off with DoIP UDP discovery.
- UI language: English only; revisit multi-language in M3.
- User hasn't reported back the actual `dotnet test` output after
  pulling M2.0 — the M1.6 bundle will exercise the same tree, so
  a single test run after this bundle covers both.

## Known caveats still unresolved

- DoIP response-pending handling loops inside the outer timeout
  without bounding pending-count.
- `ReadDtcs` unconditionally enters extended session; some ECUs
  reject. Will add default-session fallback once we hit one.
- No security-access UI; API exists, UI lands M4.

## Sandbox signing note

Commits in the scratch repo at `/tmp/bench-mercedes` are unsigned
because the sandbox signing server rejects writes from that path
(`missing source`). Every M0..M2.0..M1.6 commit follows the same
pattern. Signatures re-apply when the user pushes from his Mac.
