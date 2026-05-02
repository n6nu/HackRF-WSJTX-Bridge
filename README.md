# HackRF WSJT-X Bridge — Releases

Windows installer downloads for the **HackRF WSJT-X Bridge** — a Qt6
C++ application that turns a HackRF One into a USB-SSB software
transceiver for **WSJT-X** (FT8 / FT4) and a wideband Q65 IQ source
for **QMAP**. Primarily built for 2 m and 23 cm EME on VHF / UHF /
microwave bands; the DSP is band-agnostic.

Author: **Andreas Junge, N6NU** &lt;<n6nu@arrl.net>&gt;.

---

## Latest release — v1.0.0 (first stable)

Download: **[hackrf-wsjtx-1.0.0-setup.exe](hackrf-wsjtx-1.0.0-setup.exe)**

Promoted out of beta. Verified end-to-end on 2 m for narrowband
(FT8 via WSJT-X sound-card path) and wideband (Q65 via QMAP
Linrad path) operation against real on-air signals on a
GPSDO-locked HackRF One.

Full per-version notes, feature list, system requirements and
known limitations are in [`RELEASE_NOTES.md`](RELEASE_NOTES.md).

### First-launch SmartScreen warning

The installer is **not code-signed** and is **64-bit only**
(Windows 10 / 11 x64). On first launch on a fresh machine you will
see:

> Windows protected your PC.
> Microsoft Defender SmartScreen prevented an unrecognized app from
> starting.

Click **More info → Run anyway** to proceed. You should only have
to do this once per binary. The same warning may appear once on the
installed `hackrf-wsjtx.exe` itself; handle it the same way.

### What you'll need to install separately

- **HackRF One USB driver (WinUSB)** — the installer offers to launch
  Zadig at the end to set this up automatically. Skip if `hackrf_info`
  on your machine already prints firmware/serial without an error.
- **VB-Audio Virtual Cable** — <https://vb-audio.com/Cable/>. Provides
  the `Line 1` / `Line 2` virtual sound devices the bridge uses to
  hand audio between itself and WSJT-X.
- **WSJT-X 2.7+** — for FT8 / FT4 / Q65. Configure radio as
  *Hamlib NET rigctl*, CAT network server `localhost:4533`, PTT method
  *CAT*, sound output *Line 2*, sound input *Line 1*.
- **QMAP 0.6+** *(optional, for Q65 wideband)* — typically lives at
  `C:\WSJT\wsjtx\bin\qmap.exe`. Set Network input = enabled, UDP
  port = 50004. Launch order: **bridge → WSJT-X → QMAP**.

---

## Command-line options

Double-clicking the installed shortcut launches the GUI. From a
terminal you can also pass flags. The most useful ones for testers:

| Flag | What it does |
|---|---|
| `--help` | Print the full list of options and exit. |
| `--console` | Open a separate debug-console window with the bridge's full `stderr` log (banner, audio I/O, Linrad packet stats, I/Q balance estimator state, etc.). Closing the console quits the bridge. |
| `--no-gui` | Headless mode — no main window, just the CAT server on TCP 4533, the Linrad server on TCP 49812 / UDP 50004, and audio I/O running in the background. Exits if no HackRF is connected (nothing to do without hardware). Combine with `--console` to see the log on a server-style box. |
| `--freq <Hz>` | Initial dial frequency in Hz (e.g. `--freq 144174000`). Overrides the saved INI value for this run. |
| `--linrad-gain <dB>` | Override the Linrad/QMAP digital output gain (`-20`…`+40` dB, default `+20`). Useful for loud-signal testing where the default gain causes clipping spurs. |

Example — headless run with full logging:

```
"C:\Program Files\HackRF WSJT-X Bridge\hackrf-wsjtx.exe" --no-gui --console
```

The full set of flags (HackRF gains, IF offset, PPM, audio device
overrides, test-tone modes, etc.) is in `--help`.

---

## Reporting

Send reports directly to **<n6nu@arrl.net>**. Useful
information to include:

- HackRF serial / firmware version (`hackrf_info`)
- Windows version / build
- What you tried, what you expected, what you saw
- The bridge log: relaunch with `hackrf-wsjtx.exe --console`,
  reproduce, then copy/paste the console output
- For QMAP issues, also `qmap.ini` and a screenshot of the wideband
  waterfall

---

## License

Copyright (C) 2026 Andreas Junge, N6NU &lt;<n6nu@arrl.net>&gt;.
Licensed under the **GNU General Public License version 3 or later**;
see [`LICENSE`](LICENSE).

This program is distributed in the hope that it will be useful, but
**WITHOUT ANY WARRANTY**; without even the implied warranty of
**MERCHANTABILITY** or **FITNESS FOR A PARTICULAR PURPOSE**. Use it
at your own risk.

Bundled third-party components — including libhackrf (GPLv2),
FFTW3 (GPLv2+), Qt 6 (LGPLv3), SoXR (LGPLv2.1), libusb (LGPLv2.1+),
FFmpeg shared libraries (LGPLv2.1+), and Zadig (GPLv3, by Pete
Batard / libwdi) — are documented in
[`THIRD_PARTY_LICENSES.md`](THIRD_PARTY_LICENSES.md). Source code for
the bridge itself is available on request from N6NU under the GPLv3
"written offer" provision (§6) at the email address above; a public
source-code repository will be linked here once the project leaves
beta.
