# DigiPi Configuration Reference

All user-configurable settings live in `/home/pi/localize.env`.  
The initialization script (`localize.sh`) reads this file and substitutes
these values into all relevant config files.

> **Important:** After the first initialization, only edit the `NEW*` variables.
> The `OLD*` variables are used by `localize.sh` to find and replace the
> previous values.  If you re-run `localize.sh`, update the `OLD*` values to
> match what was previously in the `NEW*` fields.

---

## Variable Reference

### Station Identity

| Variable | Example | Description |
|---|---|---|
| `NEWCALL` | `W1ABC` | Base callsign (no SSID). Applied to all config files. |
| `OLDCALL` | `KX6XXX` | Previous callsign value to be replaced |

SSIDs are appended automatically by each mode:

| SSID | Mode |
|---|---|
| `-2` | TNC / Digipeater |
| `-4` | AX.25 Node |
| `-5` | Zork game node |
| `-10` | Winlink RMS |

---

### Winlink / APRS Passwords

| Variable | Example | Description |
|---|---|---|
| `NEWWLPASS` | `MyWLPass` | Winlink CMS account password |
| `OLDWLPASS` | `XXXXXX` | Previous Winlink password |
| `NEWAPRSPASS` | `12345` | APRS-IS passcode ([generate](https://n5dux.com/ham/aprs-passcode/)) |
| `OLDAPRSPASS` | `12345` | Previous APRS passcode |
| `NEWNODEPASS` | `abc123` | AX.25 node access password (uronode) |
| `OLDNODEPASS` | `abc123` | Previous node password |

---

### Location

| Variable | Example | Description |
|---|---|---|
| `NEWGRID` | `DM79hu` | 6-character Maidenhead grid square ([find yours](https://www.levinecentral.com/ham/grid_square.php)) |
| `OLDGRID` | `CN99mv` | Previous grid square |
| `NEWLAT` | `39.1234` | Latitude in decimal degrees |
| `OLDLAT` | `39.9999` | Previous latitude |
| `NEWLON` | `-121.4567` | Longitude in decimal degrees (negative = West) |
| `OLDLON` | `-140.9999` | Previous longitude |
| `NEWGPS` | `ttyACM1` | GPS serial device (used in `/etc/default/gpsd`) |
| `OLDGPS` | `ttyACM1` | Previous GPS device |

---

### Radio Interface

| Variable | Example | Description |
|---|---|---|
| `NEWRIGNUMBER` | `3085` | Hamlib rig number, or `GPIO`, `DTR`, `RTS`, `CM108` |
| `OLDRIGNUMBER` | `3085` | Previous rig number |
| `NEWDEVICEFILE` | `ttyACM0` | Serial device file (without `/dev/`) for CAT/PTT control |
| `OLDDEVICEFILE` | `ttyACM0` | Previous device file |
| `NEWBAUDRATE` | `115200` | Serial baud rate for CAT control (`rigctld`) |
| `OLDBAUDRATE` | `115200` | Previous baud rate |

#### `NEWRIGNUMBER` values

| Value | PTT Method | Notes |
|---|---|---|
| `GPIO` | GPIO 12 via libgpiod | Default when no USB audio adapter is found |
| `DTR` | Serial port DTR line | Set `NEWDEVICEFILE` to the serial port |
| `RTS` | Serial port RTS line | Set `NEWDEVICEFILE` to the serial port |
| `CM108` | CM108 USB audio chip HID | Set `NEWDEVICEFILE` to the HID device |
| Numeric (e.g. `3085`) | Hamlib rigctld | Set `NEWDEVICEFILE` and `NEWBAUDRATE` |

Find your Hamlib rig number: `rigctl -l | grep <radio-name>`

---

### Display

| Variable | Example | Description |
|---|---|---|
| `NEWDISPLAYTYPE` | `st7789` | TFT display driver: `st7789`, `ili9341`, `ili9486`, or leave blank |
| `OLDDISPLAYTYPE` | `st7789` | Previous display type |
| `NEWBIGVNC` | *(set to any value)* | If set, changes VNC resolution from 1024×768 to 1280×1024 |

See [Displays](displays.md) for wiring details.

---

### Audio

| Variable | Example | Description |
|---|---|---|
| `NEWI2CAUDIO` | `fepi` | I²S audio HAT: `fepi` (Fe-Pi Audio) or `aiz`/`drapizero` (AudioInjector WM8731) |

If a USB audio adapter is detected at runtime, it is automatically used in
preference to the onboard or I²S audio.

---

### Optional / Advanced

| Variable | Example | Description |
|---|---|---|
| `NEWFLRIG` | *(set to any value)* | If set, enables `flrig` rig control in the VNC startup |

---

## Runtime-Readable Variables

The following variables are read dynamically at service start time (not just
during initialization), so you can change them in `localize.env` and simply
restart the service to apply:

- `NEWRIGNUMBER`
- `NEWDEVICEFILE`
- `NEWDISPLAYTYPE`
- `NEWBAUDRATE`
- `NEWLAT`
- `NEWLON`

---

## Example: Changing Your Callsign After Initialization

1. Edit `/home/pi/localize.env`, setting `NEWCALL` to your new callsign and
   `OLDCALL` to your current callsign.
2. Remount the filesystem read-write: `sudo remount`
3. Run the localization script: `cd /home/pi && ./localize.sh`
4. Reboot or restart running services.

---

## Files Modified by `localize.sh`

| Config File | Variables Applied |
|---|---|
| `/etc/ax25/uronode.conf` | `NEWCALL` |
| `/etc/ax25/uronode.perms` | `NEWCALL`, `NEWNODEPASS` |
| `/etc/ax25/nrports` | `NEWCALL` |
| `/etc/ax25/axports` | `NEWCALL` |
| `/etc/ax25/ax25d.conf` | `NEWCALL` |
| `/home/pi/direwolf.*.conf` | `NEWCALL`, `NEWAPRSPASS`, `NEWLAT`, `NEWLON`, `NEWDEVICEFILE` |
| `/home/pi/config/pat/config.json` | `NEWCALL`, `NEWWLPASS`, `NEWGRID` |
| `/usr/local/etc/rmsgw/*.xml` | `NEWCALL`, `NEWWLPASS`, `NEWGRID` |
| `/home/pi/config/WSJT-X.ini` | `NEWCALL`, `NEWGRID` |
| `/home/pi/config/JS8Call.ini` | `NEWCALL`, `NEWGRID` |
| `/home/pi/fldigi/fldigi_def.xml` | `NEWCALL` |
| `/home/pi/config/LinPac/macro/init.mac` | `NEWCALL` |
| `/home/pi/config/aprsd/aprsd.conf` | `NEWCALL`, `NEWAPRSPASS`, `NEWLAT`, `NEWLON` |
| `/etc/default/gpsd` | `NEWGPS` |
| `/etc/systemd/system/rigctld.service` | `NEWRIGNUMBER`, `NEWDEVICEFILE`, `NEWBAUDRATE` |
| `/boot/firmware/config.txt` | `NEWI2CAUDIO` |
| `/etc/tightvncserver.conf` | `NEWBIGVNC` |
| `/home/pi/vnc/xstartup` | `NEWFLRIG` |
