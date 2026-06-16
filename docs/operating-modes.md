# DigiPi Operating Modes

DigiPi supports many amateur radio digital modes.  Each mode is implemented as
a **systemd service** that can be started from the web interface, physical
buttons, or the command line.

Only one mode that uses the radio's audio (TNC, Digipeater, Tracker, etc.) can
run at a time.  Starting a new audio mode automatically stops any conflicting
service.

---

## APRS TNC (igate)

**Service:** `tnc`  
**Script:** `/home/pi/direwolf.tnc.sh`  
**Config template:** `/home/pi/direwolf.tnc.conf`  
**Web URL:** `http://digipi/direwatch.png` (live display image)

The TNC mode starts Direwolf as a full APRS igate.  It:

- Receives 1200-baud AX.25 packets from RF
- Gates received packets to the APRS-IS internet server (`noam.aprs2.net`)
- Sends a periodic position beacon to APRS-IS
- Displays heard stations on the attached TFT display via `direwatch.py`
- Bridges the KISS interface (port 8001) to Bluetooth (`rfcomm0`), so an
  Android or iOS APRS app can connect wirelessly

### PTT detection at startup

The startup script reads `localize.env` and dynamically enables the correct
PTT line in the Direwolf config at `/run/direwolf.tnc.conf`:

| `NEWRIGNUMBER` | PTT method activated |
|---|---|
| `GPIO` or no USB audio | `PTT GPIOD /dev/gpiochip0 12` |
| `DTR` | `PTT /dev/NEWDEVICEFILE DTR` |
| `RTS` | `PTT /dev/NEWDEVICEFILE RTS` |
| `CM108` | `PTT CM108 /dev/NEWDEVICEFILE` |
| Hamlib number | `PTT RIG NEWRIGNUMBER /dev/NEWDEVICEFILE` |

---

## APRS Digipeater

**Service:** `digipeater`  
**Script:** `/home/pi/direwolf.digipeater.sh`  
**Config template:** `/home/pi/direwolf.digipeater.conf`

The Digipeater mode configures Direwolf as a WIDE1-1 digipeater plus igate.  It:

- Digipeats any packet heard on RF that has `WIDE1-1` in the path
- Gates filtered internet packets back to RF (message packets only)
- Filters out abusive stations and prevents self-digipeating
- Limits internet→RF traffic to 20 packets/minute, 60 packets/5-minutes

Key configuration directives:

```
DIGIPEAT 0 0 ^WIDE1-1$  ^WIDE1-1$
IGTXVIA 0 WIDE1-1,WIDE2-1
FILTER 0 0 ! d/KX6XXX-2 & ! b/KX6XXX-2
FILTER IG 0 ( t/m & ! g/BLN* )
IGFILTER  t/m/KX6XXX-2/160  b/KX6XXX
IGTXLIMIT 20 60
```

---

## APRS Tracker

**Service:** `tracker`  
**Script:** `/home/pi/direwolf.tracker.sh`  
**Config template:** `/home/pi/direwolf.tracker.conf`

The Tracker mode uses a connected GPS receiver (via GPSD) to send
time-stamped position beacons over RF every 5 minutes via WIDE1-1.

```
GPSD localhost
TBEACON delay=00:15 every=5:00 symbol="igate" overlay=T via=WIDE1-1 ...
```

Configure the GPS device in `/etc/default/gpsd` (or via `NEWGPS` in
`localize.env`):

```bash
DEVICES="ttyACM1"   # or ttyUSB0, etc.
```

---

## HF TNC — 300 Baud (AX.25 on HF)

**Service:** `tnc300b`  
**Script:** `/home/pi/direwolf.tnc300b.sh`  
**Config template:** `/home/pi/direwolf.tnc300b.conf`

This mode configures Direwolf with a 300-baud HF modem using 1600/1800 Hz tones
suitable for use on HF amateur radio frequencies.

```
MODEM 300  1600:1800
```

Use this mode with an HF transceiver for long-distance APRS or Winlink connections.

---

## Winlink RMS Gateway

**Service:** `winlinkrms`  
**Script:** (managed by systemd, runs `rmsgw`)  
**Config:** `/usr/local/etc/rmsgw/channels.xml`, `gateway.conf`, `sysop.xml`

The Winlink RMS mode runs a full Linux RMS gateway using `rmsgw`.  It allows
other stations to send and receive email via the Winlink network through your
station.

Key configuration files:

| File | Purpose |
|---|---|
| `/usr/local/etc/rmsgw/channels.xml` | Channel definition (frequency, callsign, baud) |
| `/usr/local/etc/rmsgw/gateway.conf` | Gateway callsign, grid, sysop info |
| `/usr/local/etc/rmsgw/sysop.xml` | Sysop registration details |
| `/usr/local/etc/rmsgw/banner` | Banner shown to connecting stations |
| `/etc/ax25/axports` | AX.25 port (`radio KX6XXX-10 1200 255 2 VHF`) |
| `/etc/ax25/ax25d.conf` | Routes `[KX6XXX-10 VIA radio]` to `rmsgw` |

---

## AX.25 Node

**Service:** `node`  
**Script:** `/home/pi/direwolf.node.sh`  
**Config template:** `/home/pi/direwolf.node.conf`

Runs Direwolf as a 1200-baud packet TNC connected to `uronode`, providing a
full AX.25 packet node.  Stations can connect to `KX6XXX-4` over RF and access
the node's services.

Node configuration: `/etc/ax25/uronode.conf`, `/etc/ax25/uronode.perms`

```
[KX6XXX-4 VIA radio]
default  * * * * * * - root /usr/sbin/uronode uronode
```

---

## ARDOP

**Service:** `ardop`  
**Script:** `/home/pi/ardop.sh`

Runs `piardopc` — an HF sound-modem for the ARDOP protocol.  ARDOP is used
primarily with the Pat Winlink client for HF email connections.

Web interface: `http://digipi:8515`

---

## Pat (Winlink Email Client)

**Script:** `/home/pi/pat.sh`

Starts the Pat Winlink email client web server.  If no radio modem is already
running, `pat.sh` automatically starts the AX.25 node service first.

Access Pat at `http://digipi:8080` (forwarded through Apache at `http://digipi/pat` if configured).

---

## FT8 / WSJTX

**Service:** `wsjtx`  
**Script:** `/home/pi/wsjtx.sh`

Starts WSJT-X inside a VNC desktop session and serves it via noVNC so you
can operate from any web browser.

Access: `http://digipi:6080` or `http://digipi/ft8`

The CPU governor is set to `performance` for best timing accuracy.

---

## JS8Call

**Service:** `js8call`  
**Script:** `/home/pi/js8call.sh`

Starts JS8Call inside a VNC session, served via noVNC.

Access: `http://digipi/js8`

---

## FLDigi

**Service:** `fldigi`  
**Script:** `/home/pi/fldigi.sh`

Starts FLDigi inside a VNC session, served via noVNC.

Access: `http://digipi/fld`

---

## SSTV

**Service:** `sstv`  
**Script:** `/home/pi/sstv.sh`

Starts a slow-scan television application inside a VNC session.

Access: `http://digipi/tv`

---

## WebChat

**Service:** `webchat`  
**Script:** `/home/pi/webchat.sh`

Provides a browser-based APRS message chat interface.

Access: `http://digipi/webchat.php`

---

## LinPac

**Script:** `/home/pi/linpac.sh`

Terminal-based AX.25 packet email and bulletin client.  Run from a terminal
or SSH session.

---

## Zork

An easter egg — connecting to `KX6XXX-5` via AX.25 over RF drops the connecting
station into the classic Zork text adventure game, served by `/home/pi/zork.sh`.

---

## Starting / Stopping Services

```bash
# Start a mode
sudo systemctl start tnc

# Stop a mode
sudo systemctl stop tnc

# Check status
sudo systemctl status tnc

# View logs
journalctl -u tnc -f
```

Or use the web interface at `http://digipi`.
