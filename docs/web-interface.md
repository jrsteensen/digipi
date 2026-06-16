# DigiPi Web Interface

DigiPi includes a PHP web control panel served by Apache on port 80.
Access it at `http://digipi` (mDNS hostname) or `http://<pi-ip-address>`.

---

## Pages

### Main Dashboard (`/`)

`index.php` — The main control panel.  Shows the current status of all
services and provides toggle buttons to start or stop each mode.

**Modes you can control from the dashboard:**

| Button | Service |
|---|---|
| APRS TNC | `tnc` |
| APRS TNC HF (300b) | `tnc300b` |
| APRS Tracker | `tracker` |
| APRS Digipeater | `digipeater` |
| WebChat | `webchat` |
| AX.25 Node | `node` |
| Winlink Server | `winlinkrms` |
| WSJTX / FT8 | `wsjtx` |
| JS8Call | `js8call` |
| SSTV | `sstv` |
| FLDigi | `fldigi` |
| ARDOP | `ardop` |
| Pat (Winlink client) | `pat` |

Each button POSTs to `index.php` with the service name and an `on`/`off`
payload, which calls `sudo systemctl start/stop <service>`.

---

### Initialization (`/setup.php`)

First-run configuration form.  See [Getting Started](getting-started.md) for
the full walkthrough.  Accepts:

- Callsign, passwords, grid square, latitude/longitude
- GPS device, AX.25 node password
- Default boot mode
- Display type, radio rig number, device file, baud rate
- VNC screen size option
- flrig enable option
- Audio HAT selection

On submit, it writes values to `/home/pi/localize.env` and calls
`/home/pi/localize.sh` to apply them across all config files.

---

### Live APRS Display (`/direwatch.php` and `/direwatch.png`)

- `/direwatch.png` — PNG snapshot of the TFT display, updated whenever a new
  packet is heard.  Served directly from `/run/direwatch.png`.
- `/direwatch.php` — Auto-refreshing page that embeds the PNG.

---

### GPS Status (`/gps.php`)

Shows current GPS fix status from `gpsd`.

---

### System Information (`/sysinfo.php`)

Shows CPU temperature, load average, memory usage, disk space, and uptime.

---

### System Log (`/syslog.php`)

Tails the system journal for DigiPi-related service messages.

---

### Direwolf Log (`/log.php`)

Tails `/home/pi/direwolf.log` in the browser.

---

### WiFi Configuration (`/wifi.php`)

Manage WiFi networks from the browser using `wpa_cli`.

---

### ALSA Audio (`/alsa.php`)

Shows detected audio devices and levels.

---

### AX.25 Call (`/axcall.php`)

Web-based AX.25 connect interface.

---

### Web Chat (`/webchat.php`)

APRS message chat interface.

---

### FT8 / WSJTX (`/ft8/`)

Embedded noVNC session for WSJTX.

---

### JS8Call (`/js8/`)

Embedded noVNC session for JS8Call.

---

### FLDigi (`/fld/`)

Embedded noVNC session for FLDigi.

---

### SSTV (`/tv/`)

Embedded noVNC session for the SSTV application.

---

### Shell (`/shell.php`)

⚠️ **Security note:** This page provides a web-based terminal shell (via
`ttyd`).  It is intended for local network use only.  Do not expose the
DigiPi web interface to the public internet.

---

## Accessing GUI Applications via noVNC

FLDigi, WSJTX, JS8Call, and SSTV run inside a VNC desktop and are served via
noVNC so you can operate them from any browser without installing a VNC client.

Direct noVNC access:

```
http://digipi:6080/vnc.html?host=digipi&port=6080&autoconnect=true
```

The VNC password is `test11` (set in the VNC startup configuration).

---

## Network Ports Summary

| Port | Service |
|---|---|
| 80 | Apache web server (DigiPi control panel) |
| 6080 | noVNC WebSocket proxy (GUI apps) |
| 8000 | Direwolf AGWPE interface |
| 8001 | Direwolf KISS TCP interface |
| 8080 | Pat Winlink HTTP server |
| 8515 | ARDOP `piardopc` modem |
| 5901 | VNC server (internal) |
