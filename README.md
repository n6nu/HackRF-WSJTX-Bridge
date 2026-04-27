# HackRF WSJT-X Bridge — Beta Releases

Windows installer downloads for the **HackRF WSJT-X Bridge** — a Qt6
C++ application that turns a HackRF One into a USB-SSB software
transceiver for **WSJT-X** (FT8 / FT4) and a wideband Q65 IQ source
for **QMAP**. Primarily built for 2 m and 23 cm EME on VHF / UHF /
microwave bands; the DSP is band-agnostic.

Author: **Andreas Junge, N6NU** &lt;<n6nu@arrl.net>&gt;.

---

## Latest beta — v0.99.1

Download: **[hackrf-wsjtx-0.99.1-setup.exe](hackrf-wsjtx-0.99.1-setup.exe)**

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
