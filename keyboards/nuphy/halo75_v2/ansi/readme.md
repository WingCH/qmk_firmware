# NuPhy Halo75 V2

*NuPhy Halo75 V2 is a 75% wireless mechanical keyboard with Mac Globe dual-role key support.*

![NuPhy Halo75 V2](https://nuphy.com/products/halo75-v2)

* Keyboard Maintainer: [nuphy](https://github.com/nuphy-src)
* Hardware Supported: NuPhy Halo75 V2 PCB
* Hardware Availability: [NuPhy](https://nuphy.com/products/halo75-v2)

## Features

This implementation includes special support for macOS Globe key functionality:

- **MAC_GLOBE**: Custom keycode that mimics the Apple Globe key for input source switching
- **Globe/Ctrl Dual-Role**: Uses QMK's `LCTL_T(KC_NO)` with custom tap handling - tap for Globe, hold for Ctrl

## Make examples

Make example for this keyboard (after setting up your build environment):

    make nuphy/halo75_v2/ansi:default

Flashing example for this keyboard:

    make nuphy/halo75_v2/ansi:default:flash

For VIA support:

    make nuphy/halo75_v2/ansi:via

## VIA Setup

To use the Globe/Ctrl dual-function key in VIA:

1. Flash the `:via` firmware
2. Open VIA and load your keyboard layout
3. Go to **SPECIAL** → **ANY**
4. Enter keycode: **0x2100** (this is `LCTL_T(KC_NO)`)
5. Drag it to your desired key position

## Documentation

For detailed implementation information, see:
- [Halo75 V2 VIA firmware 刷入指引（繁體中文）](../../../../docs/halo75_v2_via_flashing.md)
- [Mac Globe Dual-Role Documentation (中文)](../../../../docs/mac_globe_dual_role.md)
- [Mac Globe Dual-Role Documentation (English)](../../../../docs/mac_globe_dual_role_en.md)

## Bootloader

Enter the bootloader in one way:

* **Bootmagic reset**: Hold down the key at (0,0) in the matrix (usually the top left key or Escape) and plug in the keyboard

See the [build environment setup](https://docs.qmk.fm/#/getting_started_build_tools) and the [make instructions](https://docs.qmk.fm/#/getting_started_make_guide) for more information. Brand new to QMK? Start with our [Complete Newbs Guide](https://docs.qmk.fm/#/newbs).