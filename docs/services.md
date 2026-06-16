# DigiPi Systemd Services Reference

DigiPi uses systemd to manage all operating modes.  Service unit files are
installed in `/etc/systemd/system/` (sourced from `systemd/system/` in this
repository).

---

## Service Overview

| Service | Description | Conflicts with |
|---|---|---|
| `digipi-boot` | Runs at startup; shows online banner and optionally starts a default mode | — |
| `tnc` | APRS igate (1200 baud VHF) | All other audio modes |
| `tnc300b` | APRS HF TNC (300 baud, 1600/1800 Hz) | All other audio modes |
| `digipeater` | APRS WIDE1-1 digipeater + igate | All other audio modes |
| `tracker` | GPS APRS tracker | All other audio modes |
| `node` | AX.25 packet node (uronode) | All other audio modes |
| `winlinkrms` | Winlink RMS gateway (rmsgw) | All other audio modes |
| `ardop` | ARDOP HF modem (piardopc) | All other audio modes |
| `wsjtx` | WSJT-X FT8/FT4 (via VNC) | All other VNC modes |
| `js8call` | JS8Call (via VNC) | All other VNC modes |
| `fldigi` | FLDigi multi-mode TNC (via VNC) | All other VNC modes |
| `sstv` | Slow-scan television (via VNC) | All other VNC modes |
| `webchat` | APRS web chat | — |
| `pat` | Pat Winlink email client | — |
| `rigctld` | Hamlib radio control daemon | — |
| `autohotspot` | WiFi hotspot fallback | — |
| `digipi-resolv` | DNS resolver helper | — |

---

## Service Details

### `digipi-boot`

```ini
[Unit]
Description=DigiPi initial boot service
After=default.target

[Service]
Type=simple
ExecStartPre=sleep 10
ExecStart=/home/pi/online.sh
Restart=no

[Install]
WantedBy=default.target
```

Runs 10 seconds after boot to ensure the network is up, then calls
`/home/pi/online.sh` which displays the IP address on the TFT display.

To auto-start a mode at boot, uncomment the appropriate `ExecStart` line and
re-run `systemctl daemon-reload`.  Only one `ExecStart` can be active at a time.

---

### `tnc`

```ini
[Unit]
After=graphical.target

[Service]
ExecStartPre=+systemctl stop fldigi sstv wsjtx ardop tnc300b digipeater node winlinkrms js8call tracker
ExecStart=/home/pi/direwolf.tnc.sh
User=pi
WorkingDirectory=/home/pi/
```

Stops all conflicting services before starting the TNC.  Runs as the `pi` user.

---

### `digipeater`

Similar to `tnc`.  Runs `/home/pi/direwolf.digipeater.sh`.

---

### `tracker`

Similar to `tnc`.  Runs `/home/pi/direwolf.tracker.sh`.  Requires a GPS
receiver connected and `gpsd` running.

---

### `tnc300b`

Similar to `tnc`.  Runs `/home/pi/direwolf.tnc300b.sh` with 300-baud HF modem.

---

### `node`

Runs `/home/pi/direwolf.node.sh`, starting Direwolf as a TNC and attaching
`uronode` for AX.25 node functionality.

---

### `winlinkrms`

Runs the Winlink RMS gateway.  Requires `rmsgw` to be installed and
configured in `/usr/local/etc/rmsgw/`.

---

### `ardop`

Runs `/home/pi/ardop.sh` which starts `piardopc` on port 8515.

---

### `wsjtx` / `js8call` / `fldigi` / `sstv`

Each of these starts a VNC desktop session and launches the application inside
it, then starts the noVNC proxy on port 6080 for browser access.

---

### `rigctld`

Hamlib `rigctld` daemon for CAT control.  Configuration (rig number, device,
baud rate) is set by `localize.sh` in the service unit file.

---

### `autohotspot`

When no known WiFi network is found, creates a WiFi hotspot so you can connect
directly to the Pi.

- SSID: `DigiPi`
- Default hotspot IP: `10.0.0.5`

---

## Common systemctl Commands

```bash
# Start a service
sudo systemctl start tnc

# Stop a service
sudo systemctl stop tnc

# Restart a service
sudo systemctl restart tnc

# Enable a service to start at boot
sudo systemctl enable tnc

# Disable a service from starting at boot
sudo systemctl disable tnc

# Check service status
sudo systemctl status tnc

# View service logs (follow)
journalctl -u tnc -f

# Reload systemd after editing unit files
sudo systemctl daemon-reload
```

---

## Setting a Default Boot Mode

Edit `/etc/systemd/system/digipi-boot.service` and uncomment the `ExecStart`
line for your preferred default mode:

```ini
[Service]
ExecStartPre=sleep 10
ExecStart=/home/pi/online.sh        # always runs — shows IP on display
#ExecStart=systemctl start tnc      # uncomment ONE of these
#ExecStart=systemctl start digipeater
#ExecStart=systemctl start tracker
#ExecStart=systemctl start node
#ExecStart=systemctl start winlinkrms
#ExecStart=systemctl start wsjtx
#ExecStart=systemctl start js8call
#ExecStart=systemctl start fldigi
#ExecStart=systemctl start sstv
```

Then reload:

```bash
sudo systemctl daemon-reload
sudo reboot
```

> Alternatively, use the **Initialization** web page to set the default mode;
> it edits this file automatically.
