# Getting Started with DigiPi

This guide walks you through setting up DigiPi on a Raspberry Pi — from flashing the SD card through first-boot localization.

---

## Hardware Requirements

| Component | Recommendation |
|---|---|
| Raspberry Pi | Pi 3B, 3B+, 4, or Zero 2W |
| MicroSD Card | 8 GB minimum, Class 10 / A1 |
| Power Supply | 5V 2.5A (Pi 3) or 5V 3A (Pi 4) |
| Audio interface | USB audio adapter (preferred) or 3.5mm TRRS |
| Radio | Any VHF/UHF or HF transceiver with audio + PTT |
| Display (optional) | ST7789 1.3", ILI9341 2.8", or ILI9486 3.5" TFT |
| Buttons (optional) | Two momentary push-buttons |

---

## Option 1 — Flash the Pre-built Image (Recommended)

1. Download the DigiPi SD card image from <https://digipi.org/>  
   *(Patreon supporters at <http://patreon.com/KM6LYW> get access.)*
2. Flash it to a microSD card using one of:
   - [Raspberry Pi Imager](https://www.raspberrypi.com/software/) — easy GUI, available for Windows / macOS / Linux
   - [Balena Etcher](https://etcher-docs.balena.io/) — cross-platform alternative
   - `dd` — command-line on Linux/macOS
3. Insert the card, connect your Pi to your network, and power on.
4. Open `http://digipi` (or the Pi's IP address) in a browser.
5. Complete the **Initialization** form — see [First Boot](#first-boot) below.

> **16 GB microSD cards** are the sweet spot for DigiPi — the image is small
> and DigiPi minimizes SD card writes to extend the card's life.

---

## Option 2 — Manual Setup from This Repository

This path is for developers who want to build from source or understand how DigiPi works.

### Prerequisites

- Raspberry Pi OS (Bookworm or later, 32-bit or 64-bit)
- The following packages installed:

```bash
sudo apt update
sudo apt install -y direwolf ax25-tools ax25-apps \
    python3-gpiod python3-pil python3-numpy python3-dotenv \
    python3-adafruit-circuitpython-rgb-display \
    apache2 php tightvncserver novnc \
    fldigi wsjtx js8call pat
```

### Copy Repository Files

```bash
git clone https://github.com/jrsteensen/digipi.git
cd digipi

# Copy configs (review before overwriting your own)
sudo cp -r etc/ax25/*      /etc/ax25/
sudo cp    systemd/system/* /etc/systemd/system/
sudo cp -r usr/local/bin/* /usr/local/bin/
sudo cp -r usr/local/etc/* /usr/local/etc/
sudo cp -r var/www/html/*  /var/www/html/
cp -r home/pi/.  /home/pi/

sudo systemctl daemon-reload
sudo systemctl enable digipi-boot
```

---

## First Boot

### 1. Open the Initialization Page

Navigate to `http://digipi` (or `http://<pi-ip-address>`) and click **Initialization**.

### 2. Fill in Your Details

| Field | Description |
|---|---|
| **Callsign** | Your base callsign, no SSID (e.g. `W1ABC`) |
| **Winlink Password** | Your Winlink CMS password — [create an account](https://www.winlink.org/user) |
| **APRS Password** | Your APRS-IS passcode — [generate one](https://n5dux.com/ham/aprs-passcode/) |
| **Grid Square** | 6-character Maidenhead grid (e.g. `DM79`) — [find yours](https://www.levinecentral.com/ham/grid_square.php) |
| **Latitude** | Decimal degrees (e.g. `39.1234`) |
| **Longitude** | Decimal degrees, negative = West (e.g. `-121.4567`) |
| **GPS Device** | Serial device for external GPS (e.g. `ttyACM1`, `ttyUSB0`) |
| **AX.25 Node Password** | Any alphanumeric string for your packet node |
| **Default Mode** | Service to start automatically at boot |
| **Display Type** | `st7789`, `ili9341`, `ili9486`, or leave blank for none |
| **Radio Rig Number** | Hamlib rig number, `GPIO`, `DTR`, `RTS`, or `CM108` |
| **Device File** | Serial port for radio control (e.g. `ttyACM0`, `ttyUSB0`) |
| **Baud Rate** | CAT control baud rate (e.g. `9600`, `115200`) |

### 3. Click Initialize

DigiPi runs `/home/pi/localize.sh`, which performs a global search-and-replace
across all configuration files to substitute your details for the placeholder
values (default callsign `KX6XXX`).

> **Note:** Initialization is designed to run once.  Running it a second time
> requires updating the `OLD*` values in `/home/pi/localize.env` to match your
> current settings.

### 4. Reboot

```bash
sudo reboot
```

---

## Subsequent Configuration Changes

After the first initialization, most settings can be changed by editing
`/home/pi/localize.env` directly and restarting the relevant service:

```bash
sudo nano /home/pi/localize.env   # edit the NEW* values
sudo systemctl restart tnc        # or whichever service is running
```

See [Configuration Reference](configuration.md) for a full list of variables.

---

## Network Access

| Service | URL / Port |
|---|---|
| Web control panel | `http://digipi` or `http://<ip>` |
| Direwolf AGWPE | `<ip>:8000` |
| Direwolf KISS | `<ip>:8001` |
| Bluetooth KISS | `/dev/rfcomm0` → port 8001 |
| noVNC (GUI apps) | `http://digipi:6080` |
| Pat (Winlink email) | `http://digipi/pat` (port 8080 internally) |
| WSJT-X / FT8 ADI logs | `http://digipi/logs/` |

> **Tip:** You can also retrieve WSJT-X `.adi` log files over the network with
> `scp`:
> ```bash
> scp pi@<ip>:~/.local/share/WSJT-X/wsjtx_log.adi ~/Desktop/
> ```
> or use an SFTP client such as FileZilla.

---

## Editing Files on the Pi

DigiPi runs with a **read-only filesystem** by default to protect the SD card.
Before editing any system file, make the filesystem writable:

```bash
sudo remount
```

Rebooting returns the filesystem to read-only mode.  Always shut down cleanly
(via the web interface shutdown button or `sudo shutdown now`) rather than
pulling the power while the filesystem is writable, to avoid corruption.

---

## Saving Your Configuration

The `saveconfigs.sh` script in `/home/pi/` backs up your current configuration
files so they survive a reflash or SD card replacement:

```bash
/home/pi/saveconfigs.sh
```

---

## Initialization Checklist Template

Copy and customize this checklist to speed up upgrades and re-installations:

```
============================
DigiPi Initialization Screen
============================

* First boot creates a PtP WiFi network: SSID DigiPi / password abcdefghij
* **Change the default WiFi password as soon as possible after first boot!**
* Browse to 10.0.0.5  (digipi.local may also work, but the raw IP is reliable)
* Go to WiFi Setup and connect to your home network before continuing
* Be careful installing large additional packages without first expanding the
  filesystem — see https://github.com/KD9YQK/digipi_plus/blob/main/extend_filesystem.md

Callsign:         {yours}
Winlink Password: {yours}
APRS Password:    {yours}
Grid Square:      {yours}
Latitude:         {yours}
Longitude:        {yours}
GPS Device:       ttyACM1
AX.25 Node Pass:  abc123
Screen Type:      ILI9486 320x480
Enable FLRig:     {unchecked}
Large Display:    {checked}
Radio Interface:  DigiRig Mobile

Post-init checklist:
* Local login is pi / raspberry — change this password after first boot
* "sudo remount" to make the filesystem writable before editing files
* If using the ILI9486 display and the image is upside-down, issue
  "sudo remount" and edit digibanner.py and direwatch.py to change
  LOWER_RIGHT to UPPER_LEFT
* Adjust audio levels for your radio and interface via the Audio menu
* If running more than one DigiPi simultaneously, rename each one:
  "sudo remount", edit /etc/hostname and /etc/hosts, run a GUI app
  once (fixes .Xauthority), then reboot
```
