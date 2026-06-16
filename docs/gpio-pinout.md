# DigiPi GPIO Pinout Reference

This document describes how DigiPi uses the Raspberry Pi GPIO header.

---

## Raspberry Pi 40-Pin GPIO Header

Pin numbers are physical (board) numbers.  BCM numbers are used in software.

```
                    3V3  [ 1] [ 2] 5V
   SDA1 / GPIO  2  [ 3] [ 4] 5V
   SCL1 / GPIO  3  [ 5] [ 6] GND
         GPIO  4*  [ 7] [ 8] GPIO 14 / TXD
               GND [ 9] [10] GPIO 15 / RXD
         GPIO 17  [11] [12] GPIO 18 / PCM_CLK
         GPIO 27  [13] [14] GND
         GPIO 22  [15] [16] GPIO 23 **
               3V3 [17] [18] GPIO 24 **
 MOSI / GPIO 10  [19] [20] GND
 MISO / GPIO  9  [21] [22] GPIO 25 **
 SCLK / GPIO 11  [23] [24] GPIO  8 / CE0
               GND [25] [26] GPIO  7 / CE1
   SDA0 / GPIO  0  [27] [28] GPIO  1 / SCL0
         GPIO  5  [29] [30] GND
         GPIO  6  [31] [32] GPIO 12 **
         GPIO 13  [33] [34] GND
         GPIO 19  [35] [36] GPIO 16 **
         GPIO 26  [37] [38] GPIO 20
               GND [39] [40] GPIO 21
```

`**` = pins used by DigiPi (see table below)  
`*`  = SPI chip-select CE0 used by displays (physical pin 7 / BCM GPIO 4)

---

## DigiPi Pin Assignments

| BCM GPIO | Physical Pin | Direction | DigiPi Function | Notes |
|:---:|:---:|:---:|---|---|
| **4**  | 7  | Output | SPI Chip-Select (CS/CE0) | ST7789 and ILI9341 displays only |
| **8**  | 24 | Output | SPI Chip-Select (CE0) | ILI9486 display only (hardware SPI CE0) |
| **10** | 19 | Output | SPI MOSI | Display data |
| **9**  | 21 | Input  | SPI MISO | (not used by displays, reserved by SPI bus) |
| **11** | 23 | Output | SPI SCLK | Display clock |
| **12** | 32 | Output | **PTT** (Push-to-Talk) | Active-high; drives radio TX |
| **16** | 36 | Output | **DCD** (Data Carrier Detect) | Drives green LED; set by Direwolf |
| **23** | 16 | Input  | **Button 1 — TNC toggle** | Pull-up; button to GND |
| **24** | 18 | Input  | **Button 2 — Digipeater toggle** | Pull-up; button to GND; _repurposed as ILI9486 DC_ |
| **25** | 22 | Output | Display DC (Data/Command) | ST7789 and ILI9341 only; _repurposed as ILI9486 RST_ |

**Display-specific GPIO usage summary:**

| GPIO | ST7789 / ILI9341 | ILI9486 |
|:---:|---|---|
| 4  | CS (chip-select) | — |
| 8  | — | CS / CE0 (hardware SPI) |
| 24 | Button 2 (Digipeater) | **DC** (Data/Command) |
| 25 | **DC** (Data/Command) | **RST** (Reset) |

> **ILI9486 note:** When using the large 3.5″ ILI9486 display, GPIO 24 is
> repurposed as the display DC pin and GPIO 25 as the RST pin.  This
> conflicts with the Digipeater button, so physical push-buttons are **not
> available** when the ILI9486 display is installed.  Use the web interface
> to control modes instead.

---

## PTT (GPIO 12)

Direwolf drives GPIO 12 high when transmitting.  Connect this pin to your
radio's PTT line through an appropriate interface circuit (optocoupler or
direct connection depending on your radio's requirements).

Configuration in every Direwolf template:

```
#PTT GPIOD /dev/gpiochip0 12
```

The startup scripts (`direwolf.tnc.sh`, `direwolf.digipeater.sh`, etc.) uncomment
the correct PTT line at runtime based on `NEWRIGNUMBER` in `localize.env`.

Supported PTT methods (set via `NEWRIGNUMBER`/`NEWDEVICEFILE`):

| `NEWRIGNUMBER` value | PTT method |
|---|---|
| `GPIO` (or no USB audio) | GPIO 12 via libgpiod |
| `DTR` | Serial port DTR line |
| `RTS` | Serial port RTS line |
| `CM108` | USB CM108 audio adapter HID |
| Numeric Hamlib rig number | Hamlib `rigctld` |

---

## DCD / Carrier Detect LED (GPIO 16)

Direwolf drives GPIO 16 high when it detects a signal on the audio input.
Connect a green LED (with appropriate resistor) between GPIO 16 and GND to
provide a visual carrier-detect indicator.

Configuration in every Direwolf template:

```
DCD GPIOD /dev/gpiochip0 16
```

---

## Push-Buttons (GPIO 23 and GPIO 24)

DigiPi supports two optional momentary push-buttons managed by `digibuttons.py`
(started by the `digipi-boot` systemd service).

| GPIO | Physical Pin | Function |
|:---:|:---:|---|
| 23 | 16 | Toggle TNC on/off (starts/stops the `tnc` systemd service) |
| 24 | 18 | Toggle Digipeater on/off (starts/stops the `digipeater` service) |

Wire each button between the GPIO pin and GND.  The pins are configured with
internal pull-ups and 10 ms software debounce via `libgpiod`.

---

## SPI Display Interface

DigiPi uses the hardware SPI0 bus for the TFT display.

| Signal | BCM GPIO | Physical Pin |
|---|:---:|:---:|
| MOSI  | 10 | 19 |
| MISO  | 9  | 21 |
| SCLK  | 11 | 23 |
| CE0 (CS) — ST7789 / ILI9341 | 4 | 7 |
| CE0 (CS) — ILI9486 | 8 | 24 |
| DC — ST7789 / ILI9341 | 25 | 22 |
| DC — ILI9486 | 24 | 18 |
| RST — ILI9486 | 25 | 22 |

Enable SPI in `/boot/firmware/config.txt` (or via `raspi-config`):

```
dtparam=spi=on
```

> Do **not** install the kernel framebuffer driver (`fbtft` / `notro`).
> DigiPi uses userspace Python libraries to drive the display directly.

See [Displays](displays.md) for full wiring details per display type.

---

## Audio Interface

DigiPi uses the Raspberry Pi's audio hardware (or a USB audio adapter) for
radio TX/RX audio.  **No GPIO pins are used for audio** — connect audio via:

- The Pi's 3.5 mm TRRS jack (audio out + mic in), **or**
- A USB audio adapter (preferred — avoids ground-loop issues), **or**
- An I²S audio HAT (e.g. Fe-Pi Audio, AudioInjector)

### Recommended USB Audio Adapter Wiring

```
Radio mic/speaker jack ←→ USB audio adapter ←→ USB port on Pi
```

The startup scripts automatically detect and prefer a USB audio device
over the onboard audio when one is present.

---

## Reference Schematic (Conceptual)

```
Raspberry Pi                        Radio
─────────────────────────────────────────────────
GPIO 12 (PTT) ──[optocoupler]──────► PTT line
GPIO 16 (DCD) ──[330Ω]──[LED]──GND  (green LED)
GPIO 23       ──[button]──GND        (TNC button)
GPIO 24       ──[button]──GND        (Digi button)

Audio OUT (3.5mm or USB) ──────────► Radio mic/audio in
Audio IN  (3.5mm or USB) ◄──────── Radio speaker/audio out

SPI (GPIO 4,10,11,25) ─────────────► TFT Display
```
