# Bringing the Mac Globe Dual-Role Key to the NuPhy Halo75 V2

## Why bother?

I love the NuPhy Halo75 V2, but two things bugged me when hacking on its QMK firmware:

1. The stock framework/VIA config doesn’t expose the macOS “Globe” key.
2. The left Control key feels wasted on its own. I wanted a dual-role key that taps as Globe (for input source switching) and holds as Control for shortcuts.

The solution was to add two custom keycodes:

- `MAC_GLOBE`: faithfully mimics the Apple Globe key.
- `MAC_GLOBE_CTRL`: tap for Globe, hold for Control – a true hybrid.

## Overview of changes

| File | Purpose |
|------|---------|
| `keyboards/nuphy/halo75_v2/ansi/ansi.h` | Define `MAC_GLOBE` and `MAC_GLOBE_CTRL` in the custom keycode enum |
| `keyboards/nuphy/halo75_v2/ansi/ansi.c` | Implement the dual-role logic, juggling Consumer usage 0x029D and Control |
| `keyboards/nuphy/halo75_v2/ansi/keymaps/via/NuPhy Halo75 via3.json` | Expose the new keycodes in the VIA keymap |
| `keyboards/nuphy/halo75_v2/nuphy-halo75-v2-via.json` | Provide an official VIA definition with the extra keys |

## QMK implementation highlights

### Custom keycode definitions

```c
enum custom_keycodes {
    ...
    MAC_GLOBE,
    MAC_GLOBE_CTRL,
    ...
};
```

They are anchored at `QK_KB_0`, so VIA writes and reads the correct 16-bit values.

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

### `MAC_GLOBE_CTRL`

The dual-role logic fires Globe immediately on press, then promotes to Control if another key joins the party.

```c
static bool     mac_globe_ctrl_pressed    = false;
static bool     mac_globe_ctrl_mod_active = false;
static bool     mac_globe_ctrl_tapped     = false;

bool process_record_kb(uint16_t keycode, keyrecord_t *record) {
    if (mac_globe_ctrl_pressed && !mac_globe_ctrl_mod_active) {
        if (keycode != MAC_GLOBE_CTRL && record->event.pressed) {
            if (mac_globe_ctrl_tapped) {
                host_consumer_send(0);
                mac_globe_ctrl_tapped = false;
            }
            register_code(KC_LCTL);
            mac_globe_ctrl_mod_active = true;
        }
    }

    switch (keycode) {
        case MAC_GLOBE_CTRL:
            if (record->event.pressed) {
                mac_globe_ctrl_pressed    = true;
                mac_globe_ctrl_mod_active = false;
                mac_globe_ctrl_tapped     = true;
                host_consumer_send(0x029D); // instant Globe feedback
            } else {
                if (mac_globe_ctrl_mod_active) {
                    unregister_code(KC_LCTL);
                } else if (mac_globe_ctrl_tapped) {
                    host_consumer_send(0); // pure tap: release Globe
                }
                mac_globe_ctrl_pressed    = false;
                mac_globe_ctrl_mod_active = false;
                mac_globe_ctrl_tapped     = false;
            }
            return false;
    }
    return true;
}
```

> Tip: We emit the 0x029D consumer usage on press to keep tap latency near-zero. If a second key appears, we cancel the consumer report and switch to Control.

## VIA support

Both VIA JSON files list the new entries under `customKeycodes`:

```json
{
    "name": "Mac\nGlobe",
    "title": "Mac Globe"
},
{
    "name": "Mac\nGlbCtrl",
    "title": "Mac Globe Ctrl"
}
```

Reload the layout in VIA and you can drag these keys onto any position.

## Build and flash

```bash
qmk compile -kb nuphy/halo75_v2/ansi -km via
```

Flash the resulting `nuphy_halo75_v2_ansi_via.bin`, then try the three scenarios on macOS:

| Action | Result |
|--------|--------|
| Tap `Mac Globe` | Input source menu pops instantly |
| Tap `Mac Globe Ctrl` | Same as above |
| Hold `Mac Globe Ctrl` + C | Sends Control+C |
| Hold `Mac Globe Ctrl` alone | Releases the Globe usage, no stray Control |

## Takeaways

- QMK’s built-in `MT()`/`LT()` macros are great, but Consumer usages (like 0x029D) need bespoke handling.
- Firing the Globe usage on press keeps the interaction snappy; just remember to cancel it when you transition to Control.
- Commenting the state machine up front saves future-me (or future-you) lots of head scratching.

## What’s next?

1. Commit `nuphy-halo75-v2-via.json` so others can load the VIA profile as-is.
2. Extend the idea: add custom keycodes for Spotlight, Siri, media controls—anything macOS exposes via Consumer usages.

If you’ve been craving a true Mac Globe experience on the Halo75 V2, these two keycodes deliver it with a sprinkle of dual-role magic.
