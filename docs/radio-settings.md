# DigiPi Radio and Interface Settings

The following combinations and settings have been reported as successful by
DigiPi users and are provided as good starting points.

Also see the main Direwolf user guide for detailed configuration options:
<https://github.com/wb2osz/direwolf/blob/master/doc/User-Guide.pdf>

---

## Yaesu FTM-3100R and DigiRig Mobile

*Scott/KE8SAU suggests the following:*

- **Radio:** Turn the FTM-3100R speaker volume to 75% (3 o'clock position)
- **Alsamixer:** From DigiPi's main menu select `Audio` to open Alsamixer.
  Press `F5`, set speaker to −4 dB, mic to −20 dB, capture to −9 dB, 00 AGC.
  Exit and click `Save Configuration` on the DigiPi main menu.

---

## Yaesu FTM-400DR and DigiRig Mobile

- Craig_AZ reports that the "DigiRig Mobile Cable for Yaesu MiniDin10
  9600-baud Data and CAT" from Amazon works, but audio levels may be a bit
  high by default — adjust Alsamixer inputs accordingly.
- LeeMcC/KI5YPR: The DigiRig cable for an older Yaesu FT2600M attaches to the
  9600 pin (discriminator output, not audio).  The output level was higher than
  Direwolf expects (~200); cutting a trace in the DigiRig to reduce gain by
  20 dB brought it down to ~25, which most APRS stations report cleanly.  See
  DigiRig's online documentation for details.

---

## Icom IC-705

*Craig/KM6LYW suggests the following:*

- **Radio:** Disable noise reduction, notch, and noise blanker (`[Function]`
  key); use FM-D mode; check Setup → Connectors and set USB to "AF"
  (audio frequency, not IF).

---

## Xiegu G90 and DigiRig Mobile

*Bob/KD9YQK suggests the following:*

- **WSJT-X / JS8Call / flrig — Transceiver tab:** Select your serial port.
  Baud: 19200, 1 stop bit.  Uncheck Echo, RTS/DTS, and the two 12 V boxes.
  Set 2 retries, 2 timeout, 2 CMDs, 200 ms poll, 0 byte interval.
- **PTT tab:** Select CAT, Data/Pkt, and Fake it.
- If using flrig, select FLRig as the rig in JS8Call, WSJT-X, QSSTV, etc.

---

## Yaesu FT-991

*Craig/KM6LYW suggests the following menu settings:*

| Menu # | Name | Value |
|---|---|---|
| 028 | GPS/232C SELECT | RS232C |
| 029 | 232C RATE | 38400 bps |
| 031 | CAT RATE | 38400 bps |
| 038 | MIC SCAN RESUME | PAUSE |
| 062 | DATA MODE | OTHERS |
| 064 | OTHER DISP (SSB) | 1500 Hz |
| 065 | OTHER SHIFT (SSB) | 1500 Hz |
| 066 | DATA LCUT FREQ | OFF |
| 068 | DATA HCUT FREQ | OFF |
| 072 | DATA PORT SELECT | USB |
| 073 | DATA OUT LEVEL | 50 |
| 075 | FM OUT LEVEL | 100 |
| 077 | FM PKT PORT SELECT | USB |
| 078 | FM PKT TX GAIN | 4 |

Video walkthrough: <https://www.youtube.com/watch?v=MwygYBBgCcw&t=1129s>

---

## Yaesu FT-857D and DigiRig Mobile

*Scott/KE8SAU suggests the following:*

- **Radio:** The FT-857D supports CAT control; configure it for digital modes.
  WC0O's video covers the setup: <https://www.youtube.com/watch?v=zVUVYwo0Eh0>
- **WSJT-X / JS8Call / flrig — Transceiver tab:** Select FT-857D and your
  serial port.  Baud must match the radio (9600 is typical), 1 stop bit, no
  parity, no flow control, no power-up line.
- **PTT tab:** Select CAT, Data/Pkt, and Fake it.  If using flrig, select
  FLRig as the rig in other applications.
- **Direwolf PTT override** (add to your `direwolf.*.conf`):
  ```
  PTT RIG 1022 {your serial port}
  ```
- *Minitechnicus:* The FT-857 / FT-817 also works with the DRA HAT using a
  mini-DIN 6 cable — no configuration changes needed.  The same cable and HAT
  work with Kenwood TM-V71, ICOM 208H, and Alinco 135.

---

## Yaesu FT-817 / FT-818 and DigiRig Mobile

*K9BDH suggests the following:*

- **Radio:** Supports CAT control; configure for digital modes.
  KM1NDY's guide: <https://km1ndy.com/yaesu-818-digital-modes-wsjt-x-digirig-setup/>
- **WSJT-X / JS8Call / flrig — Transceiver tab:** Select FT-817 or FT-818 and
  your serial port.  Baud must match the radio (38400 if following KM1NDY's
  guide), default stop bit, default handshake.
- **PTT tab:** Select RTS, Data/Pkt, and Fake it.  If using flrig, select
  FLRig as the rig in other applications.
- **Direwolf PTT override** (add to your `direwolf.*.conf`):
  ```
  PTT RIG 1020 {your serial port}
  ```
- *Minitechnicus:* See FT-857D note above — same cable and HAT work here too.

---

## General HT (Handy-Talkie) Tips

HT operation with DigiPi is possible but requires adjusted expectations:

- Strongly consider an **AIOC (All-In-One Cable)** with a separate antenna.
  Rubber duck antennas can freeze the AIOC through its vertical USB cable.
  In WSJT-X and JS8Call, select the `plughw` entry for the AIOC.
- **DigiRig interfaces** also work with HTs — they cost a bit more but can be
  reused with other radios via appropriate cables.
- **VOX is strongly discouraged** — triggering PTT with VOX is too slow for
  reliable digital modes.
- **UV5R-style radios** are hit-or-miss due to poor receiver audio quality.
  Yaesu, Icom, Kenwood, AnyTone, and Tidradio HTs perform better.  Cheap HTs
  always have lesser receiver quality, which can seriously hurt digital-mode
  receive performance.
- Disable squelch, turn off roger beeps, and tweak settings to remove the
  squelch tail.

---

## Tidradio H3/H8 and AIOC

*Scott/KE8SAU suggests the following:*

- **Radio:** Turn speaker volume just under full; disable squelch.
- **Alsamixer:** From DigiPi's main menu select `Audio`, press `F5`:
  - `AIOC Audio In` → 70
  - `AIOC Audio In 1` → 70
  - `AIOC Audio Out Volume` → 84
  - `AIOC Audio Out Volume 1` → 84
  - Exit and click `Save Configuration`.
- Add `ARATE 48000` to your `direwolf.*.conf` files.
- In WSJT-X and JS8Call, select the `plughw` entry for the AIOC.
- Use an external antenna — transmitting through a rubber duck may cause the
  AIOC to freeze or reset.

---

## Yaesu FT-710 (and possibly FTDX10)

The FT-710 produces crackling/popping audio with a Raspberry Pi at 48000 Hz
sample rates.  All audio must be forced to 44100 Hz.

```bash
sudo remount

# /usr/share/alsa/alsa.conf — change the dmix rate:
sudo nano /usr/share/alsa/alsa.conf
# Set:  defaults.pcm.dmix.rate 44100

# ~/.asoundrc — force 44100 and configure ARDOP:
nano ~/.asoundrc
```

```
pcm.ARDOP {
   type plug
   slave.pcm "dmix"
}

# force 44100
pcm.!default {
type rate
slave {
       pcm "plughw:0,0"
       rate 44100
      }
}
```

```bash
# direwolf.tnc300b.conf — append 38400 to the PTT RIG line:
nano direwolf.tnc300b.conf
# Change:  #PTT RIG RIGNUMBER DEVICEFILE
# To:      #PTT RIG RIGNUMBER DEVICEFILE 38400

# ardop.sh — use "default" device:
nano ardop.sh
# Set:  cd /tmp && /usr/local/bin/piardopc 8515 default ARDOP
```
