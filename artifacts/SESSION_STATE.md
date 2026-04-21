# Session state — resume here next time

## Where we are — M2.4 shipped (adapter + protocol selector UI)

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
| 713d67e | M2.1   | Fix ISO-TP CS4007 (ReadOnlyMemory across awaits)          |
| 7c0b04f | M2.2   | Drop unused Hal using in IsoTpChannelTests                |
| 20881c6 | M2.3   | KWP2000 client (ISO 14230) + 11 KWP tests                 |
| 0495c83 | M2.4   | Adapter + protocol selector UI, transport generalization  |

## What M2.4 added

- **`IDiagnosticSession`** (`src/MercedesDiag.Services/IDiagnosticSession.cs`)
  shared contract for UDS and KWP sessions, plus `DtcEntry(Code, Status, Raw)`
  record struct that normalises DTCs for the UI.
- **`KwpDiagnosticSession`** — wraps `KwpClient`, enters
  `KwpSession.ExtendedDiagnostics` before read/clear, clears group `0xFF00`.
- **`DiagnosticSession`** (UDS) now implements `IDiagnosticSession` and returns
  `IReadOnlyList<DtcEntry>` instead of UDS-specific records.
- **`ConnectionService`** rewritten:
  - New enums `AdapterTransport { DoipEnet, Pcan, J2534, Sdconnect }` and
    `DiagnosticProtocol { Uds, Kwp }`.
  - `ConnectionSettings` is a flat record with nullable per-transport fields:
    `DoipEndpoint`, `CanBitrateKbps` (500), `CanTxId` (0x7E0), `CanRxId`
    (0x7E8), `CanExtendedFrames`, `PcanChannel` ("PCAN_USBBUS1"),
    `J2534DllPath`, `J2534DeviceIndex`, `SdconnectHost`.
  - `ConnectAsync` dispatches on transport: DoIP → `DoipChannel`;
    PCAN/J2534 → `IsoTpChannel` over the matching CAN adapter;
    SDconnect → `NotSupportedException` ("lands in later milestone").
  - Takes three `ILogger<T>` in the ctor (added `KwpDiagnosticSession`).
    Generic-host DI resolves it automatically — no `Program.cs` change.
- **`MainWindowViewModel`**:
  - `AdapterTransports` and `Protocols` read-only lists for ComboBox binding.
  - `SelectedTransport` / `SelectedProtocol` observable properties.
  - Derived flags `IsDoipTransport` / `IsCanTransport` / `IsPcanTransport` /
    `IsJ2534Transport` / `IsSdconnectTransport` drive panel visibility via
    `[NotifyPropertyChangedFor]`.
  - New input properties: `CanBitrateKbps`, `CanTxIdHex`, `CanRxIdHex`,
    `CanExtendedFrames`, `PcanChannel`, `J2534DllPath`, `J2534DeviceIndex`,
    `SdconnectHost`.
  - `TryBuildSettings` helper validates per-transport inputs and returns a
    human-readable `error` on failure.
  - `DtcRowViewModel` now takes `DtcEntry` (protocol-agnostic).
- **`MainWindow.axaml`**:
  - Sidebar widened to 360px and wrapped in a `ScrollViewer`.
  - Adapter + Protocol ComboBoxes at top.
  - Conditional `<StackPanel IsVisible="{Binding IsXxxTransport}">` sub-panels
    for DoIP (IP+port), CAN (bitrate+TX+RX+extended), PCAN (channel +
    "hardware backend lands in M2.5"), J2534 (DLL path+device +
    "Windows-only"), SDconnect (host + "not implemented yet").

## Next session — first actions

User on his Mac:
```
cd /Users/mohammedouchrif/Bench-mercedes
git pull ~/normalize.css/artifacts/bench-mercedes-m2.3-to-m2.4.bundle main
git push origin main
dotnet restore
dotnet build
dotnet test
dotnet run --project src/MercedesDiag.App
```

(If the Mac is missing the earlier increments, pull them in order first:
`m1.6-to-m2.1fix`, `m2.1fix-to-m2.2fix`, `m2.2fix-to-m2.3`, then this one.)

Expected after pull:
- Sidebar now has **Adapter** and **Protocol** dropdowns.
- Selecting a transport reveals that transport's settings panel:
  DoIP → IP+port; CAN (PCAN/J2534) → bitrate+TX+RX+extended; PCAN adds
  channel; J2534 adds DLL path+device; SDconnect shows "not implemented".
- **32 tests passing** (21 from M2.0/M1.6 + 11 new KWP tests).
- DoIP round-trip still works exactly as before (default values unchanged).

## Next milestone — M2.5 (PCAN-USB P/Invoke backend)

Goal: real CAN hardware on Windows/Linux. macOS gets a stub that throws
`PlatformNotSupportedException` — PCAN-USB has no macOS driver.

- `src/MercedesDiag.Hal/Pcan/PcanAdapter.cs` — replace the throwing stub
  with P/Invoke to `PCANBasic.dll` (Windows) / `libpcanbasic.so` (Linux).
- Bitrate mapping (Mercedes buses are almost always 500 kbps; diagnostic
  buses on newer cars sometimes 1 Mbps; interior CAN-B 83 kbps).
- Extended-frame + listen-only flags.
- Frame-level tests against a fake channel + manual bench test with a
  PCAN-USB on Windows.

## Then — M2.6 (J2534) and M2.7 (SDconnect passthrough)

- `src/MercedesDiag.Hal/J2534/J2534Adapter.cs` — P/Invoke to the vendor
  DLL selected by `J2534DllPath`. Windows-only in v1.
- SDconnect is complex enough (proprietary MUX protocol over TCP) that it
  stays out until a real user need surfaces.

## Open items

- Sandbox scope for `rohite1983/Bench-mercedes` still not granted,
  so the bundle workflow continues.
- Vehicle IP default `169.254.0.1` still unverified against the
  user's actual ENET cable — if it doesn't come up, we'll switch
  Connect to kick off with DoIP UDP discovery.
- UI language: English only; revisit multi-language in M3.
- User will test the new UI **tomorrow at the workshop** with the
  ENET cable; the bundle covers the UI + protocol plumbing he needs
  for that test.

## Known caveats still unresolved

- DoIP response-pending handling loops inside the outer timeout
  without bounding pending-count.
- `ReadDtcs` unconditionally enters extended session; some ECUs
  reject. Will add default-session fallback once we hit one.
- No security-access UI; API exists, UI lands M4.
- PCAN/J2534 selection in the UI surfaces panels but the HAL backends
  still throw "not implemented" on `OpenAsync` — M2.5/M2.6.

## Sandbox signing note

Commits in the scratch repo at `/tmp/bench-mercedes` are unsigned
because the sandbox signing server rejects writes from that path
(`missing source`). Every M0..M2.4 commit follows the same pattern.
Signatures re-apply when the user pushes from his Mac.
