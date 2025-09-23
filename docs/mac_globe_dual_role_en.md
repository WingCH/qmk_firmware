# Bringing the Mac Globe Dual-Role Key to the NuPhy Halo75 V2

## Why bother?

I love the NuPhy Halo75 V2, but two things bugged me when hacking on its QMK firmware:

1. The stock framework/VIA config doesn’t expose the macOS “Globe” key.
2. The left Control key feels wasted on its own. I wanted a dual-role key that taps as Globe (for input source switching) and holds as Control for shortcuts.

The solution was to add one custom keycode and leverage QMK's built-in functionality:

- `MAC_GLOBE`: faithfully mimics the Apple Globe key.
- `LCTL_T(KC_NO)`: uses QMK's built-in Mod-Tap feature, tap for Globe, hold for Control – more robust and reliable.

## Overview of changes

| File | Purpose |
|------|---------|
| `keyboards/nuphy/halo75_v2/ansi/ansi.h` | Removed `MAC_GLOBE_CTRL`, simplified custom keycode definitions |
| `keyboards/nuphy/halo75_v2/ansi/ansi.c` | Uses QMK's built-in `LCTL_T(KC_NO)` for dual-role, sends Globe Consumer usage on tap |
| `keyboards/nuphy/halo75_v2/ansi/keymaps/via/NuPhy Halo75 via3.json` | Removed Mac Globe Ctrl entry, now uses standard QMK functionality |
| `keyboards/nuphy/halo75_v2/nuphy-halo75-v2-via.json` | Provide an official VIA definition with the extra keys |

## QMK implementation highlights

### Custom keycode definitions

```c
enum custom_keycodes {
    ...
    MAC_GLOBE,
    ...
};
```

They are anchored at `QK_KB_0`, so VIA writes and reads the correct 16-bit values. Removed `MAC_GLOBE_CTRL` in favor of QMK's built-in solution.

### `MAC_GLOBE`

Straightforward consumer usage emission:

```c
case MAC_GLOBE:
    if (record->event.pressed) {
        host_consumer_send(0x029D);
    } else {
        host_consumer_send(0);
    }
    return false;
```

### `LCTL_T(KC_NO)` Integration with Globe Functionality

Now using QMK's built-in Mod-Tap feature, sending Globe Consumer usage when tap behavior is detected:

```c
bool process_record_kb(uint16_t keycode, keyrecord_t *record) {
    // Handle LCTL_T(KC_NO) - the 0x2100 keycode for Globe/Ctrl dual function
    if (keycode == LCTL_T(KC_NO)) {
        if (record->tap.count && record->event.pressed) {
            // Tapped: Send Globe consumer key
            host_consumer_send(0x029D);
            return false;  // Prevent default processing
        } else if (record->tap.count && !record->event.pressed) {
            // Tap released: Cancel Globe consumer key
            host_consumer_send(0);
            return false;  // Prevent default processing
        }
        // For hold (no tap.count), let QMK handle the Ctrl modifier
        return true;
    }
    // ... other processing
}
```

> Tip: Using QMK's built-in `LCTL_T` is more stable and reliable. QMK automatically handles tap/hold detection, and we only need to send the Globe usage when a tap is detected.

## VIA support

The VIA JSON file now only includes the Globe key under `customKeycodes`:

```json
{
    "name": "Mac\nGlobe",
    "title": "Mac Globe"
}
```

For the Globe/Ctrl dual-function key, we now use QMK's built-in `LCTL_T(KC_NO)` (keycode: 0x2100), which VIA displays as "LCtl_T(KC_NO)".

## Build and flash

```bash
qmk compile -kb nuphy/halo75_v2/ansi -km via
```

Flash the resulting `nuphy_halo75_v2_ansi_via.bin`, then try the three scenarios on macOS:

| Action | Result |
|--------|--------|
| Tap `Mac Globe` | Input source menu pops instantly |
| Tap `LCTL_T(KC_NO)` | Same as above |
| Hold `LCTL_T(KC_NO)` + C | Sends Control+C |
| Hold `LCTL_T(KC_NO)` alone | Only acts as Control, no Globe signal |

## VIA Setup

To use the Globe/Ctrl dual-function key in VIA:

1. Open VIA and load your keyboard layout
2. Go to the **SPECIAL** tab
3. Select **ANY**
4. Enter the keycode: **0x2100**
5. Drag it to your desired key position

### How 0x2100 is calculated

The keycode `0x2100` comes from QMK's `LCTL_T(KC_NO)` macro:

- `LCTL_T()` creates a Mod-Tap key with Left Control as the modifier
- `KC_NO` (0x00) is the tap keycode (no key)
- QMK's Mod-Tap keycodes start at `0x2000`
- Left Control modifier adds `0x0100`
- Therefore: `0x2000` + `0x0100` + `0x00` = `0x2100`

## Takeaways

- Initially tried a custom dual-role implementation, but later found that QMK's built-in `LCTL_T` combined with custom tap handling is more stable and reliable.
- Using `record->tap.count` accurately detects QMK-recognized tap behavior, avoiding the complexity of implementing custom tap/hold detection.
- Consumer usages (like 0x029D) still need special handling, but integrating them into QMK's Mod-Tap system works much better.
- The code becomes cleaner and more maintainable.

## What’s next?

1. Commit `nuphy-halo75-v2-via.json` so others can load the VIA profile as-is.
2. Extend the idea: add custom keycodes for Spotlight, Siri, media controls—anything macOS exposes via Consumer usages.

If you’ve been craving a true Mac Globe experience on the Halo75 V2, these two keycodes deliver it with a sprinkle of dual-role magic.
