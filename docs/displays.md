# DigiPi Supported Displays

DigiPi can optionally drive a small SPI TFT display to show APRS station
activity, the current operating mode, and status indicators (PTT / DCD).

Three display families are supported, driven entirely from userspace Python
(no kernel framebuffer module required).

---

## Supported Displays

| Model | Driver | Resolution | Size | Confirmed Source | Price |
|---|---|:---:|---|---|---|
| Adafruit Mini PiTFT 1.3" | `st7789` | 240 × 240 | 1.3" | [Adafruit #4484](https://www.adafruit.com/product/4484), [Amazon](https://amazon.com/gp/product/B08F9XTKGK) | ~$15 |
| Adafruit PiTFT 2.8" | `ili9341` | 320 × 240 | 2.8" | [Adafruit #2423](https://www.adafruit.com/product/2423) | ~$45 |
| Generic 3.5" SPI TFT | `ili9486` | 480 × 320 | 3.5" | [Amazon B07V9WW96D](https://www.amazon.com/gp/product/B07V9WW96D) | ~$25 |

> ⚠️ **Supplier note:** Not all suppliers are equal — displays sourced outside
> the Americas have been reported to not work.  Stick to the confirmed links
> above for new builds.

Set `NEWDISPLAYTYPE` in `/home/pi/localize.env` to `st7789`, `ili9341`, or
`ili9486`.  Leave it blank or set to `none` if no display is attached.

---

## ST7789 — Adafruit Mini PiTFT 1.3" (240×240)

**Product:** [Adafruit #4484](https://www.adafruit.com/product/4484)

This is the default display.  It plugs directly onto the GPIO header with no
additional wiring when using the Adafruit PiTFT HAT form factor.

### Pin Connections

| Display Pin | BCM GPIO | Physical Pin | Function |
|---|:---:|:---:|---|
| MOSI | 10 | 19 | SPI data |
| SCLK | 11 | 23 | SPI clock |
| CS   | 4  | 7  | SPI chip-select |
| DC   | 25 | 22 | Data / Command |
| RST  | —  | —  | Tied to 3.3V (no reset pin needed) |
| BL   | —  | —  | Backlight (tied to 3.3V or PWM) |
| VCC  | —  | 1  | 3.3V |
| GND  | —  | 6  | GND |

### Button Connections (on HAT)

| Button | BCM GPIO | Physical Pin |
|---|:---:|:---:|
| Button 1 (TNC toggle) | 23 | 16 |
| Button 2 (Digi toggle) | 24 | 18 |

### Software Configuration

```python
dc_pin = digitalio.DigitalInOut(board.D25)   # GPIO 25
cs_pin = digitalio.DigitalInOut(board.D4)    # GPIO 4
spi    = board.SPI()

disp = st7789.ST7789(
    spi, cs=cs_pin, dc=dc_pin,
    baudrate=64000000,
    height=240, width=240, y_offset=80,
    rotation=180
)
```

### Enable SPI

```bash
sudo raspi-config   # Interface Options → SPI → Enable
```

or add to `/boot/firmware/config.txt`:

```
dtparam=spi=on
```

---

## ILI9341 — Adafruit PiTFT 2.8" (320×240)

**Product:** [Adafruit #2423](https://www.adafruit.com/product/2423)

### Pin Connections

Same as ST7789 above (DC = GPIO 25, CS = GPIO 4).

### Software Configuration

```python
dc_pin = digitalio.DigitalInOut(board.D25)
cs_pin = digitalio.DigitalInOut(board.D4)
spi    = board.SPI()

disp = ili9341.ILI9341(
    spi, cs=cs_pin, dc=dc_pin,
    baudrate=64000000,
    width=240, height=320, rotation=270
)
```

---

## ILI9486 — Generic 3.5" SPI TFT (480×320)

**Example product:** [Amazon link](https://www.amazon.com/Resistive-Screen-IPS-Resolution-Controller/dp/B07V9WW96D)

This display uses a slightly different SPI mode and requires a direct
`spidev` connection rather than the Adafruit CircuitPython stack.

> ⚠️ **GPIO 24 conflict:** The ILI9486 uses GPIO 24 as the DC (Data/Command)
> pin.  This means the physical **Button 2** (Digipeater toggle) is **not
> available** when this display is installed.  Use the web interface to control
> modes instead.

### Pin Connections

| Display Pin | BCM GPIO | Physical Pin | Function |
|---|:---:|:---:|---|
| MOSI | 10 | 19 | SPI data |
| SCLK | 11 | 23 | SPI clock |
| CS   | 8  | 24 | SPI CE0 (hardware SPI CE0) |
| DC   | 24 | 18 | Data / Command |
| RST  | 25 | 22 | Reset |
| VCC  | —  | 1  | 3.3V |
| GND  | —  | 6  | GND |

### Software Configuration

```python
from spidev import SpiDev
spi = SpiDev(0, 0)
spi.mode = 0b10
spi.max_speed_hz = 48000000

disp = ili9486.ILI9486(
    spi=spi, rst=25, dc=24,
    origin=ili9486.Origin.LOWER_RIGHT
).begin()
disp.invert()
```

The ILI9486 Python driver is included at `/home/pi/ILI9486.py`.

---

## Display Software

### `direwatch.py`

Tails the Direwolf log file and renders heard APRS stations on the display in
real time.  Also shows PTT (red) and DCD (green) status icons by monitoring
GPIO 12 and GPIO 16.

```bash
/home/pi/direwatch.py \
    --log /run/direwolf.log \
    --title_text "DigiPi TNC" \
    --lat 39.1234 --lon -121.4567 \
    --display st7789 \
    --save /run/direwatch.png
```

The `--save` flag writes a PNG copy of the screen to `/run/direwatch.png`,
which the web interface serves at `http://digipi/direwatch.png`.

The `-o` / `--one` flag switches to single-station full-screen mode, showing
one station at a time with a distance/bearing readout.

### `digibanner.py`

Displays a static splash screen (mode name, URL, IP address).  Called by mode
startup scripts to show the current mode at boot.

```bash
/home/pi/digibanner.py \
    --big "DigiPi" \
    --small "http://digipi" \
    --tiny "192.168.1.42" \
    --graphic /home/pi/km6lyw.png \
    --display st7789
```

---

## Installing Python Dependencies

```bash
pip3 install adafruit-circuitpython-rgb-display \
             adafruit-blinka \
             pillow numpy python-dotenv gpiod pyinotify aprslib
```

For the ILI9486, also install `spidev`:

```bash
pip3 install spidev
```
