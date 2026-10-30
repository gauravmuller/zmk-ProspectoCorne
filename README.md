# ProspectoCorne: ZMK config

Corne-style split keyboard (Corney Island PCB, nice!nano v2 halves) with a dongle:
Xiao BLE + Waveshare 1.69" LCD (Prospector hardware, no light sensor, own battery),
running the [YADS dongle screen](https://github.com/janpfischer/zmk-dongle-screen/tree/upgrade-4.1).

- ZMK **main** (Zephyr 4.1), pinned to commit `4dd95d7` in `config/west.yml` and
  `.github/workflows/build.yml` (keep the two in sync when updating).
- **Dongle-only:** the halves talk to the dongle, the dongle talks to the computer (USB or Bluetooth).
  Without the dongle the keyboard can't type.

## Repo layout

```
boards/shields/prospectocorne/
  Kconfig.shield                         declares prospectocorne_left / _right / _dongle
  Kconfig.defconfig                      dongle = central (2 halves + 5 BT profiles); halves = peripherals
  prospectocorne.dtsi                    halves: rows 5-8, matrix, OLED (I2C 2/3), Corne 6-column layout
  prospectocorne_left.overlay            left columns 21,20,19,18,15,14
  prospectocorne_right.overlay           right columns (reversed, offset 6)
  prospectocorne_dongle.overlay          dongle: no keys, same layout + matrix
  boards/nice_nano_nrf52840_zmk.overlay  per-key RGB: data on pin 1 (P0.06), 21 LEDs
  prospectocorne_left.conf               left half: OLED, RGB, deep sleep, debounce
  prospectocorne_right.conf              right half: same
  prospectocorne_dongle.conf             dongle: no sleep, no RGB
  prospectocorne.keymap                  keymap (all parts)
  prospectocorne.zmk.yml                 metadata: halves (pro_micro)
  prospectocorne_dongle.zmk.yml          metadata: dongle (seeed_xiao)
config/prospectocorne.conf               all parts: battery reporting, +8 dBm range, Studio
config/prospectocorne_dongle.conf        dongle screen: battery, idle timeout, brightness, orientation
config/west.yml                          ZMK main + YADS (both pinned)
build.yaml                               what GitHub builds
```

## Firmware

Download from GitHub: **Actions -> latest run -> Artifacts**.

| Part | File |
|---|---|
| Dongle (Xiao BLE) | `prospectocorne_dongle` |
| Left half | `prospectocorne_left` |
| Right half | `prospectocorne_right` |
| Clear pairings | `settings_reset_xiao_ble`, `settings_reset_nice_nano` |

To flash: double-tap the reset button (or use the **Boot** key on the Adjust layer), a USB drive
appears, copy the `.uf2` file onto it.

## First-time setup / re-pairing

1. Switch off both halves.
2. Flash `settings_reset_xiao_ble` to the dongle and `settings_reset_nice_nano` to both halves.
3. Flash `prospectocorne_dongle` to the dongle, then unplug it.
4. Flash `prospectocorne_left` and `prospectocorne_right`.
5. Plug the dongle back in, switch on the **left half first** and wait for its battery to show on
   the screen, then switch on the right half (the screen orders batteries by pairing order).
6. Use the dongle over USB, or pair it to the computer over Bluetooth (BT1-BT5 on the Adjust layer).

## Keymap

| Layer | How | What's on it |
|---|---|---|
| Base | default | QWERTY, Ctrl/Shift on the left edge, Esc bottom-right, Win/Space/Enter/Alt on thumbs |
| Lower | hold left thumb `LWR` | numbers, arrows (H J K L), Home/End/PgUp/PgDn, Del |
| Raise | hold right thumb `RSE` | symbols `! @ # $ % ^ & * ( )`, `- = [ ] \ _ + { } \|` |
| Adjust | hold `LWR` + `RSE` | F1-F12, BT1-BT5 + clear, USB/BT toggle, screen keys, RGB, media, Boot |

- **Caps Word:** press Left Shift + Esc together.
- **Dongle screen keys (Adjust):** F22 screen on/off, F23 dimmer, F24 brighter.
- **Edit without reflashing:** plug the dongle into USB and open https://zmk.studio in Chrome or Edge.
  Two spare layers are reserved for Studio.

## Dongle screen

- Shows layer, modifiers, WPM, USB/Bluetooth status, and battery for both halves + the dongle.
- Turns off after 5 minutes idle; the next keypress wakes it.
- Settings: `config/prospectocorne_dongle.conf`. Changing the layout itself needs a fork of YADS.

## Notes
- The half OLEDs show their own battery/connection; only the dongle knows the active layer.
- Halves deep-sleep after 15 minutes idle; any key wakes them. The dongle never sleeps (it has no keys).
