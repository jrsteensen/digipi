# DigiPi Tips, Hacks, and Notes

---

## Make DigiPi Services Start at Boot

Newer versions of DigiPi let you choose a default boot service from the
**Initialization** web page — it edits `digipi-boot.service` automatically.

On older versions, use the manual approach:

```bash
sudo remount
sudo nano /etc/systemd/system/digipi-boot.service
```

Uncomment exactly one `ExecStartPost` line for the mode you want to auto-start,
then reload:

```bash
sudo systemctl daemon-reload
sudo reboot
```

See [Systemd Services](services.md) for the full list of available modes.

---

## Interesting Interface Notes

### Noise Filters

Common-mode noise on USB and audio cables can cause sound card disconnections
when your radio keys up.  DigiRig cables ship with ferrite clamps, but USB
cables to other audio interfaces (AIOC, etc.) typically don't.

Wrapping cables in [ferrite snap-on cores](https://www.amazon.com/dp/B07PVKZ9PX)
can eliminate these disconnections.  Keep cables a few inches away from RF
antenna paths as well.

### Multiple DigiPi Installations

If you run more than one DigiPi on the same network, rename each one to avoid
hostname conflicts:

1. `sudo remount`
2. Edit `/etc/hostname` and `/etc/hosts` with the new name.
3. Run any GUI application (WSJT-X, etc.) once to regenerate `.Xauthority`.
4. Reboot and test.

### DigiRig Mobile vs. Lite

- The **Lite** version is essentially a modified CM108 USB sound card with PTT.
- The **Mobile** version adds CAT control capabilities.

If your radio supports CAT control, get the Mobile version.  If not, the Lite
is sufficient.  Alternatively, a $6 CM108 USB sound card can be modified to
work like a DigiRig Lite if you're comfortable with fine soldering work.

---

## Unsuitable Hardware

The following hardware has been found to be problematic or incompatible:

- **Radioddity DB20G / UV** (aka **AnyTone AT-779UV**) — Bob/KD9YQK found
  there is no dedicated PTT pin in the mic connection.  VOX is undesirable for
  digital modes, and using it would require soldering directly to the
  microphone PCB.

- **IQaudio Codec Zero** boards have been reported as problematic by several
  users.  Worth trying if you already own one, but the interfaces listed at
  <https://digipi.org> are recommended for new builds.

- **Baofeng UV-5R** style radios are hit-or-miss due to poor receiver audio
  quality.  See the [General HT Tips](radio-settings.md#general-ht-handy-talkie-tips)
  section for alternatives.

---

## Web Browser Compatibility

### Tested and Working

| Browser | Version |
|---|---|
| iPadOS / Safari | 18.0.1, 18.1.1, 18.4.1 |
| iPadOS / Brave | 1.73.97 |
| Ubuntu / Brave | 1.74.51 |
| Ubuntu / Firefox | 137.0 |
| macOS / Safari | 18.3.1 |
| macOS / Brave | 1.75.181 |
| Android / Brave | 1.62.123, 1.68.128 |
| Amazon / Silk Browser | 134.3.25.6998.207.10 |

> **Safari note:** The on/off toggle sliders don't "slide" when clicked on
> Safari — click the slider and wait about three seconds for it to move and
> turn green.

### Known Issues

- Internet Explorer and legacy Edge are not supported.

---

## Contributing

Community contributions to the documentation are very welcome!
Feel free to open a pull request against this repository with corrections,
additions, or new radio configuration examples.

The main DigiPi website is at <https://digipi.org>.
