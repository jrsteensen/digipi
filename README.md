# DigiPi

**DigiPi** is a Raspberry Pi-based amateur radio digital modes platform created by Craig Lamparter (KM6LYW).  It turns a Raspberry Pi into a self-contained software-defined radio TNC, digipeater, APRS i-gate, Winlink gateway, AX.25 node, and much more — all controllable from a web browser on your local network.

> **Get the ready-to-flash SD card image** — patrons of <http://patreon.com/KM6LYW> may download the full image at <https://digipi.org/>

This repository contains all original code and modifications to existing GPL code that ship in the DigiPi image.  It is intended for developers who want to understand the internals, build their own radio/DigiPi project, or satisfy GPL compliance requirements.

---

## Features at a Glance

| Mode | Description |
|---|---|
| **APRS TNC** | Full igate — receives and gates APRS packets to/from the internet |
| **APRS Digipeater** | WIDE1-1 RF digipeater + internet gate with message relay |
| **APRS Tracker** | GPS-driven mobile beacon via GPSD |
| **HF TNC (300 baud)** | 300-baud AX.25 on HF (1600/1800 Hz tones) |
| **Winlink RMS** | Linux RMS gateway using `rmsgw` |
| **AX.25 Node** | Full packet node running `uronode` |
| **ARDOP** | ARDOP HF modem via `piardopc` |
| **FT8 / WSJTX** | Weak-signal digital modes via browser (noVNC) |
| **JS8Call** | JS8 keyboard-to-keyboard digital mode via browser |
| **FLDigi** | Multi-mode soundcard TNC via browser |
| **SSTV** | Slow-scan television via browser |
| **WebChat** | APRS message web chat |
| **Pat** | Winlink email client |

---

## Documentation

| Document | Description |
|---|---|
| [Getting Started](docs/getting-started.md) | First-boot setup, SD card flashing, and localization |
| [GPIO Pinout](docs/gpio-pinout.md) | Raspberry Pi GPIO header and DigiPi pin assignments (PTT, DCD, LEDs, buttons) |
| [Displays](docs/displays.md) | Supported TFT displays, wiring, and supplier notes |
| [Operating Modes](docs/operating-modes.md) | All modes, scripts, and how they work |
| [Configuration Reference](docs/configuration.md) | `localize.env` variable reference |
| [Web Interface](docs/web-interface.md) | Web UI overview |
| [Systemd Services](docs/services.md) | Service reference and management |
| [Radio Settings](docs/radio-settings.md) | Community-tested radio and interface configurations |
| [Tips and Hacks](docs/tips-and-hacks.md) | Useful tips, unsuitable hardware, browser notes, and contributing |

---

## Repository Layout

```
digipi/
├── etc/ax25/          AX.25 stack configuration (axports, ax25d.conf, uronode, etc.)
├── home/pi/           User scripts, Direwolf configs, Python display drivers
├── systemd/system/    Systemd service unit files
├── usr/local/bin/     Binaries and helper scripts
├── usr/local/etc/     rmsgw, conversd, and other daemon configs
└── var/www/html/      PHP web control panel
```

---

## Quick Start (developers)

1. Flash the DigiPi image to an SD card (see <https://digipi.org/>).
2. Boot the Pi and open `http://digipi` in a browser on your network.
3. Fill out the **Initialization** page with your callsign, passwords, and location.
4. Click **Initialize**, then choose a default operating mode and reboot.

For a manual setup from this repository, see [Getting Started](docs/getting-started.md).

---

## License

Original DigiPi code is released under the **MIT License** (see individual file headers).  
Modifications to GPL components are released under the **GPL v2 or later** (see individual file headers).

Thank you for your support,  
-craig
