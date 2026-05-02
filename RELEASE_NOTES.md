# HackRF WSJT-X Bridge — Release Notes

Copyright (C) 2026 **Andreas Junge, N6NU**. Licensed under
**GNU GPL v3 or later**. See `LICENSE` (or
<https://www.gnu.org/licenses/gpl-3.0.html>). Third-party
components bundled with the installer — including libhackrf,
fftw3, Qt 6, soxr, libusb, and Zadig (Pete Batard / libwdi) — are
documented in `THIRD_PARTY_LICENSES.md`. **No warranty.**
You install and run this software at your own risk.

## v0.99.4 — spectrum waterfall toggle (2026-05-02)

The built-in spectrum / waterfall display can now be turned off from
the **View menu** (or the **Ctrl+W** shortcut). Useful when you don't
need the visual debugging and would rather not pay the CPU cost.

- View → "Show spectrum waterfall" — checkable, default on.
- New CLI flag `--no-waterfall` launches with the display off and
  persists the choice to INI key `gui/waterfall_enabled`.
- When off, three layers of work are skipped: per-IQ-buffer
  `FftEngine::pushIq()` (the per-sample int8→float + Hann window
  multiply at the full 2 Msps rate), the widget paint events, and
  the 20 Hz row-poll timer. Roughly 2–5 % of one CPU core saved.
- Default ON for first installs and for upgrades — no surprise
  change for existing testers.

Drop-in upgrade from v0.99.3; no INI migration.

## v0.99.3 — beta (2026-04-27)

- Fix: on Windows, the Settings dialog showed **IF offset = 1500**
  on a fresh install with no HackRF attached, instead of the
  documented platform default of 0. Cause: when the HackRF wasn't
  open at startup, `main.cpp` skipped `HackRfDevice::configure()`
  entirely (no hardware to talk to), so the `HackRfDevice::Config`
  struct's hardcoded 1500 fallback leaked through to the dialog
  instead of the CLI/INI-derived 0. `configure()` is now safe to
  call without an open device — it just records the requested
  config; hardware writes happen on open / Start. The Settings
  dialog now correctly reads back **0** on a fresh Windows install.

## v0.99.2 — beta (2026-04-27)

- The bridge now starts even when no HackRF is connected. Previously
  the GUI would silently fail to launch on a fresh test machine if
  the HackRF wasn't plugged in or its WinUSB driver wasn't yet
  installed. The bridge now shows a small *"HackRF not found"* dialog,
  opens the main window with the **HackRF** status row reading
  *Not connected* (amber), and lets the tester explore the Settings
  dialog and audio routing while the radio is offline. Plugging the
  HackRF in afterwards and clicking **Start** retries the open and
  recovers without a restart. Headless mode still exits with an error
  when no HackRF is found — there's nothing to do without hardware.

## v0.99.1 — beta (2026-04-27)

- INI file moved from `n6nu\VirtualSDR.ini` to `n6nu\HackRF WSJT-X
  Bridge.ini` so future bridges for other radios (RTL-SDR, AirSpy, …)
  can sit alongside without colliding on a shared INI. Existing
  installs are migrated automatically: on first launch of 0.99.1, if
  the new INI doesn't yet exist but the old one does, the bridge
  copies it across so you keep your saved gains, frequency, and
  audio device choices.

## v0.99.0 — beta (2026-04-27)

First public beta. The bridge has been verified end-to-end on 2 m for
both narrowband (FT8 via WSJT-X sound-card path) and wideband (Q65 via
QMAP Linrad path) operation, against real on-air signals and a
GPSDO-locked HackRF One.

This release goes to a small group of testers. **Please report anything
weird** — see "Reporting" at the bottom.

### Installation note (read first)

The installer **is not code-signed**. On first launch on a fresh
Windows 10/11 machine you will see this:

> **Windows protected your PC.**
> Microsoft Defender SmartScreen prevented an unrecognized app from
> starting. Running this app might put your PC at risk.

This is normal for unsigned beta software. To proceed:

1. Click **More info** in the SmartScreen dialog.
2. Click **Run anyway**.

You should only see this once per binary; Windows remembers the
decision. The same warning may appear the first time you launch the
installed `hackrf-wsjtx.exe` itself — handle it the same way.

If you are uncomfortable bypassing SmartScreen, do not install this
release. A code-signed build is on the roadmap.

### What's new in this version

- **Adaptive I/Q balance correction** on the QMAP/Linrad path. The
  HackRF's analog imbalance was leaking a residual opposite-sideband
  image at ~−20 to −30 dBc, occasionally strong enough that QMAP would
  decode the image *before* the wanted signal. A blind α/sin(φ)
  estimator now runs on the resampled IQ stream and corrects in
  software; image rejection lifts past −40 dBc on broadband content.
  Settings dialog shows live `α`, `φ`, and the dBc rejection of the
  *uncorrected* HackRF for diagnostics. (See §14 in `README.md`.)
- **Configurable Linrad output gain** — CLI `--linrad-gain <dB>`, INI
  `linrad/output_gain_db`, and a Settings spinbox (range −20 … +40 dB,
  default +20 dB). Lower this on loud test signals to avoid clipping
  spurs.
- **No more console pop-up on launch.** The exe is now a Win32-GUI
  subsystem binary; double-clicking from Explorer gives only the GUI
  window. Pass `--console` if you want a debug console with the full
  `fprintf(stderr,…)` log output.
- **QMAP wideband decoding works correctly.** Two protocol bugs were
  fixed: packets now carry all 348 IQ pairs (was 174 + zero-pad), and
  a `(-1)^n · conj(x[n])` per-sample transform compensates for QMAP's
  `FFTW_BACKWARD`-as-forward FFT direction so signals draw at the
  right frequency on the waterfall. (See §13.)

### Feature summary

#### Sound-card path (WSJT-X FT8 / FT4 / etc.)

- `rigctld`-compatible CAT TCP server on port **4533** — WSJT-X
  connects as "Hamlib NET rigctl"
- USB and LSB demodulation via Hilbert phasing (~35 dB
  opposite-sideband rejection)
- DC blocker IIR removes the HackRF's direct-conversion LO leak
- WSJT-X-style modes accepted: `USB`, `LSB`, `PKTUSB`, `PKTLSB`
- WSJT-X-driven half-duplex TX/RX with clean PTT transitions
- Carrier-leak suppression: TX is hard-stopped when PTT goes off
  (otherwise a zero-baseband stream radiates a CW carrier at the LO)

#### Wideband path (QMAP Q65)

- Linrad protocol server: TCP **49812** (parameter requests) + UDP
  **50004** (Raw16 wideband IQ stream)
- 96 kHz IQ to QMAP's `linradBuffer` format, 348 IQ pairs per 1416-byte
  packet at ~276 packets/sec
- Decimation 2 Msps → 96 kHz with separate I/Q SoXR resamplers (phase
  coherent)
- Adaptive I/Q balance correction (default-on; toggle in Settings)
- Configurable per-stream digital output gain (independent of the
  sound-card-path gain)

#### HackRF control

- LNA / VGA / AMP gains adjustable from the Settings dialog and CLI;
  changes apply live without restarting the stream
- Bias-tee toggle (manual, or PTT-driven for transverter T/R control)
- IF offset and PPM frequency correction (use 0 ppm with an external
  10 MHz reference / GPSDO)
- Platform-aware `if_offset_hz` defaults: 0 on Windows, 1500 on Linux

#### Audio I/O

- Windows: Qt Multimedia / WASAPI; works with VB-Audio Virtual Cable
  (Line 1, Line 2) for WSJT-X routing. Format negotiation (Float32 ↔
  Int16, 1ch ↔ 2ch) happens automatically.
- Linux: PipeWire via FIFO `module-pipe-sink` / `module-pipe-source`
  (named pipes `wsjtx_tx` and `wsjtx_rxfeed`)
- Software RX audio gain (0–40 dB) on Windows to compensate for
  WASAPI's lower default gain compared with PipeWire

#### GUI

- Frequency / mode / PTT / RX-stream status display
- TX and RX peak meters in dBFS, plus a single-K-calibrated dBm RX
  reading after a one-time signal-generator calibration
- 1024-bin scrolling waterfall at ~20 rows/sec
- Settings dialog with HackRF gains, IF offset, PPM, mode, audio
  device pickers, RX audio gain, Linrad output gain, and I/Q balance
  correction toggle/diagnostics
- Audio device picker auto-populates from the host's audio devices

#### Command-line modes

- `--no-gui` headless mode: no main window, just the CAT server,
  Linrad server, and audio I/O running in the background. Useful for
  server / Pi-style installs.
- `--console` (Windows): attaches a separate debug-console window to
  the otherwise-windowless GUI build, with the full `fprintf(stderr)`
  log output (audio status, Linrad packet stats, I/Q balance state,
  etc.). Combine with `--no-gui` for headless-with-log.
- All HackRF gain / freq / mode parameters can be overridden with
  individual flags (`--freq`, `--tx-gain`, `--rx-lna`, `--rx-vga`,
  `--rx-amp`, `--bias-tee`, `--if-offset`, `--ppm`, `--linrad-gain`,
  `--tx-device`, `--rx-device`). CLI overrides INI for the current
  run; clean exits write the current state back to the INI.
- `--help` lists every option.

#### Persistence

- Frequency, mode, gains, audio device names, IF offset, PPM, RX audio
  gain, Linrad gain, and I/Q balance enable state are all saved to
  `%APPDATA%\Roaming\n6nu\HackRF WSJT-X Bridge.ini`
- Last-tuned frequency is written back on every CAT change (so a
  `pkill` / Ctrl-C exit doesn't lose it)
- CLI flags override stored values for the current run; on exit,
  current values are written back

### System requirements

- **OS**: Windows 10 or 11 (x64). Tested on Windows 11 Pro build 26200.
- **Radio**: HackRF One (any HackRF with libhackrf-compatible firmware
  should work; tested on stock HackRF One firmware)
- **USB driver**: WinUSB via Zadig — the installer offers to launch
  Zadig on completion to set up the driver
- **Audio routing**: VB-Audio Virtual Cable
  (https://vb-audio.com/Cable/) — provides Line 1 and Line 2 for
  WSJT-X audio I/O. Free version is sufficient.
- **WSJT-X**: 2.7 or later for FT8/FT4. Configure radio as
  *Hamlib NET rigctl* with CAT network server `localhost:4533`,
  PTT method *CAT*, sound output *Line 2*, sound input *Line 1*.
- **QMAP**: 0.6 or later for wideband Q65 (separate install from
  WSJT-X — typically `C:\WSJT\wsjtx\bin\qmap.exe`). UDP port = 50004,
  Network input = enabled. Launch order:
  **bridge → WSJT-X → QMAP**.

### Known limitations

- **Half-duplex.** The HackRF can't transmit and receive at the same
  time, so RX is paused during TX. WSJT-X handles this transparently.
- **Single-point I/Q correction.** The new balancer compensates with
  one (α, φ) pair averaged across the 96 kHz output band. Imbalance
  on a real HackRF is mildly frequency-dependent; image rejection at
  the very edges of the band may not reach the 50+ dBc seen mid-band.
- **Single-CW-tone signals can bias the I/Q estimator.** The corrector
  clamps α to 0.5–2.0 and sin(φ) to ±0.5 so it can't blow up, but on
  unusual test signals you may see the estimator briefly drift away
  from its nominal point. With normal antenna noise + on-air signals
  this isn't an issue in practice.
- **Unsigned binary.** SmartScreen warning on first launch — see top
  of this file.
- **CAT chain required for QMAP.** QMAP only processes incoming
  Linrad UDP once it has CAT info from WSJT-X, so the bridge must be
  running with WSJT-X connected before QMAP will draw a waterfall.

### Quick start for testers

1. Run the installer. Accept the SmartScreen warning per the note above.
2. Let the installer launch Zadig and set the HackRF driver to WinUSB.
   (Skip if you already have the WinUSB driver from another HackRF
   tool — `hackrf_info` returning firmware/serial without a permission
   error is the test.)
3. Install VB-Audio Virtual Cable if you don't already have it.
4. Configure WSJT-X as described above and launch it.
5. Launch the bridge. The Settings dialog comes up minimized in a
   small window; click "Settings…" to verify gains and audio devices.
6. (Optional, for Q65 wideband) launch QMAP and verify the wide-band
   waterfall draws around the WSJT-X dial.

Detailed Windows install steps are in `WINDOWS.md`.

### Reporting

Send reports directly to Andreas Junge (N6NU) at
**<n6nu@arrl.net>**. When reporting an issue, include:

- HackRF serial / firmware version (`hackrf_info` output)
- Windows version / build
- What you tried, what you expected, what you actually saw
- The bridge log: relaunch with `hackrf-wsjtx.exe --console`, reproduce,
  then copy/paste the console output
- For QMAP-related issues, also `qmap.ini` and a screenshot of the
  wideband waterfall

### Source

This is a beta distribution. Source code is available on request from
Andreas Junge (N6NU) at **<n6nu@arrl.net>**, under the GPLv3 terms in
`LICENSE` (GPLv3 §6 "written offer"). A public source-code repository
will be linked from a future release.
