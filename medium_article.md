# QMK Implementation of Apple Globe Keycap Function

## Problem: Missing Globe Key

After buying the NuPhy Halo75 V2, I found one issue: it doesn't have Apple's Globe key. For someone like me who frequently switches between Chinese and English input methods, I'm used to this key and don't want to change my habits just because I switched keyboards.

So I wanted to find a way to use Globe key functionality on this keyboard too.

## Idea: Combine Control and Globe Functions

Thinking about it carefully, the Control key is never used alone - it always needs to be combined with other keys (Ctrl+C, Ctrl+V, etc.) to work. Meanwhile, Apple's Globe key works with just a single press to switch input methods.

Since these two functions don't conflict, I can make one key have both functions:
- Single press = Globe key (switch input method)
- Long press + other keys = Control function

## Implementation: Using QMK

QMK is an open-source keyboard firmware that lets us customize key functions. This idea can be implemented with QMK.

### Method 1: Write Logic Myself

Initially, I tried writing my own dual-function key logic, but the code was very complex and had to handle various timing issues.

### Method 2: Use QMK Built-in Features

Later, I discovered that QMK already has Mod-Tap functionality. I can use `LCTL_T(KC_NO)` as a base and add Globe functionality when detecting a single tap:

```c
bool process_record_kb(uint16_t keycode, keyrecord_t *record) {
    if (keycode == LCTL_T(KC_NO)) {
        if (record->tap.count && record->event.pressed) {
            // Detect tap: send Globe key
            host_consumer_send(0x029D);
            return false;
        } else if (record->tap.count && !record->event.pressed) {
            // Tap release: cancel Globe signal
            host_consumer_send(0);
            return false;
        }
        // Let QMK handle Control for hold cases
        return true;
    }
    // Other processing...
}
```

Benefits of this approach:
- QMK handles tap/hold detection
- I only need to handle Globe functionality
- Code is simpler

## Setup Methods

### Using VIA

For users who don't want to compile firmware themselves, you can set it up directly in VIA:

1. Open VIA and load your keyboard configuration
2. Go to **SPECIAL** → **ANY**
3. Enter keycode: **0x2100**
4. Drag it to the left Control position

**Why 0x2100?**

This number comes from QMK's keycode calculation:
- Mod-Tap base value: `0x2000`
- Left Control modifier: `0x0100`
- KC_NO (no key): `0x00`
- Total: `0x2000 + 0x0100 + 0x00 = 0x2100`

### Test Results

The experience after setup was very satisfying:

| Action | Result |
|--------|--------|
| Light press | Input method menu pops up immediately |
| Long press + C | Standard Ctrl+C copy function |
| Long press + V | Standard Ctrl+V paste function |
| Long press then release | Only Control function, no extra actions |

## Experience

The results after setup were quite good:

1. **Keep original habits**: No need to relearn new key methods
2. **No function conflicts**: Single press and long press correspond to different functions, feels natural to use
3. **Make good use of existing tools**: QMK's Mod-Tap feature is mature, much simpler than writing logic myself

For people who frequently need to switch input methods, the convenience this small change brings is quite obvious.

## Summary

This modification was relatively simple, mainly using QMK's existing features to solve real usage needs.

NuPhy Halo75 V2 + QMK's Mod-Tap feature = Globe/Control dual-function key

If you have similar needs, you can try this method.

---

**Complete code and setup instructions** can be found in my [GitHub repository](https://github.com/your-repo).

If you have similar experiences, feel free to share and discuss!

#keyboard #QMK #customization #NuPhy #experience
