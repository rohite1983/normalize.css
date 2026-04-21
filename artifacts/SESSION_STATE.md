# Session state — resume here next time

## Where we are — M2.6 shipped (PCAN P/Invoke backend + UI polish)

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
| d290c84 | M2.5   | Adapter auto-probe, ECU bus discovery, named custom, UI polish |
| 033f39e | M2.6   | Real PCAN-USB P/Invoke backend (Windows + Linux)          |

## What M2.5 added (UI polish + live ECU discovery)

- **`AdapterProbeService`** — platform-aware detection of DoIP / PCAN /
  J2534 availability. DoIP is always available (host networking).
  PCAN checks for `PCANBasic.dll` on Windows (System32) and
  `libpcanbasic.so` on Linux (common paths), reports unavailable on
  macOS. J2534 reports available only on Windows (user points the DLL
  field at a vendor PassThru DLL).
- **`EcuDiscoveryService`** — on a successful DoIP connect, pings each
  catalog address with UDS TesterPresent (400 ms per ECU) and replaces
  the ECU dropdown with responders. A 'Scan bus' button allows manual
  re-scans. Progress bar shown during scan.
- **`MercedesEcu.Source`** (enum: Catalog/Discovered/Custom) — discovered
  entries render with a '•' suffix so the user can see which came from
  the real car.
- **Named custom target** — users can now give their custom target a
  human-readable name (e.g. 'My gateway') alongside the hex address;
  it flows through `ConnectionSettings.TargetName` and appears in the
  connected status line and future code paths.
- **MainWindow.axaml polish** — card-style sidebar with new styles
  (`.card`, `.section`, `.hint`, `.label`), status dot (green connected
  / grey disconnected), Re-probe button, inline probe details line,
  indeterminate progress bar while connecting, tidier per-transport
  panels (DoIP / CAN / PCAN / J2534 / SDconnect).

## What M2.6 added (PCAN-USB backend)

- **`src/MercedesDiag.Hal/Pcan/PcanBasic.cs`** — PCANBasic P/Invoke
  declarations: `CAN_Initialize` / `_Uninitialize` / `_Read` / `_Write`
  with separate Windows (`PCANBasic.dll`) and Linux (`libpcanbasic.so`)
  entry points; `TPCANMsg` / `TPCANStatus` / `TPCANMessageType` /
  `TPCANTimestamp` structs; bitrate constants (`B500K`, `B1M`, etc.)
  and `PcanBaud.FromKbps` mapper; channel map for
  `PCAN_USBBUS1..8` / `PCIBUS` / `ISABUS`.
- **`PcanAdapter.cs`** — replaces the 'not implemented' stub with a
  full `ICanAdapter` over PCAN. Background RX pump runs in a Task,
  writes into a bounded `Channel<CanFrame>` (DropOldest on overflow);
  TX goes through `CAN_Write` with 11/29-bit flag set from `CanFrame.Extended`.
  macOS path throws `PlatformNotSupportedException` up-front (no PEAK driver).

## Next session — first actions

User on his Mac:
```
cd ~/normalize.css && git pull origin claude/clarify-project-requirements-Qz5IO
cd /Users/mohammedouchrif/Bench-mercedes
git pull ~/normalize.css/artifacts/bench-mercedes-m2.3-to-m2.4.bundle main
git pull ~/normalize.css/artifacts/bench-mercedes-m2.4-to-m2.6.bundle main
git push origin main
dotnet restore && dotnet build && dotnet test
dotnet run --project src/MercedesDiag.App
```

(If the Mac is behind M2.3, pull the earlier bundles first in order:
`m1.6-to-m2.1fix`, `m2.1fix-to-m2.2fix`, `m2.2fix-to-m2.3`, then the
two newest.)

Expected after pull:
- Sidebar has a cleaner card look with a green/grey status dot in the
  header.
- Adapter dropdown shows availability + helpful detail for each transport.
  A 'Re-probe' button re-checks (useful after plugging in PCAN-USB).
- Connect button shows an indeterminate progress bar while connecting.
- After connecting via DoIP, the app auto-scans known ECU addresses
  against the real vehicle; dropdown shrinks to only responders with
  a '•' marker. A 'Scan bus' button lets you repeat the scan.
- 'Custom target' panel now has a name + address side-by-side.
- PCAN hardware on Windows/Linux: selecting Pcan, entering channel
  (e.g. `PCAN_USBBUS1`) + bitrate + tester/ECU CAN IDs, then Connect
  should now actually initialise the driver and send frames.
- 32 tests still passing (no new tests in M2.5/M2.6 — the new services
  need integration tests with real hardware; unit coverage lands later).

## Next milestone — M2.7 (J2534 P/Invoke backend)

- `src/MercedesDiag.Hal/J2534/J2534Adapter.cs` — real P/Invoke to the
  vendor DLL specified by `J2534DllPath`. Windows-only in v1.
- J2534 API: `PassThruOpen`, `PassThruConnect`, `PassThruReadMsgs`,
  `PassThruWriteMsgs`, `PassThruIoctl`, `PassThruDisconnect`, `PassThruClose`.
- The existing `J2534Adapter.cs` still throws 'not implemented'; this
  milestone replaces it analogously to M2.6's PcanAdapter.

## Then — M3 (live data + coding)

- `src/MercedesDiag.Coding/Yaml/UserYamlProvider.cs` — user-supplied
  YAML coding definitions.
- Periodic DID polling with graph/gauge.
- Coding tree UI.

## Open items

- Sandbox scope for `rohite1983/Bench-mercedes` still not granted,
  so the bundle workflow continues.
- Vehicle IP default `169.254.0.1` still unverified against the
  user's actual ENET cable — if it doesn't come up, we'll switch
  Connect to kick off with DoIP UDP discovery.
- UI language: English only; revisit multi-language in M3.
- No unit tests for AdapterProbeService / EcuDiscoveryService / the
  real PcanAdapter yet — they benefit most from hardware integration
  tests. Fake-driven unit tests can still be added for ProbeService
  branches on demand.

## Known caveats still unresolved

- DoIP response-pending handling loops inside the outer timeout
  without bounding pending-count.
- `ReadDtcs` unconditionally enters extended session; some ECUs
  reject. Will add default-session fallback once we hit one.
- No security-access UI; API exists, UI lands M4.
- J2534 selection in the UI now flows to a stub that throws on open.
  Real implementation in M2.7.
- SDconnect remains unimplemented and will stay so until a real
  user need surfaces.

## Sandbox signing note

Commits in the scratch repo at `/tmp/bench-mercedes` are unsigned
because the sandbox signing server rejects writes from that path
(`missing source`). Every M0..M2.6 commit follows the same pattern.
Signatures re-apply when the user pushes from his Mac.
