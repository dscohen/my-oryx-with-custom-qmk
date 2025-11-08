# Combo Timing Buckets - User Guide

This keymap uses a custom **combo timing bucket system** that allows you to group combos into categories with different detection windows. This gives you fine-grained control over how quickly (or slowly) each combo detects key presses.

## Overview

By default, QMK uses a single timing value (`COMBO_TERM`) for all combos. This system lets you define multiple timing buckets, each with its own timeout value, and assign combos to buckets based on your preference.

## Quick Start

### 1. Define Timing Buckets

In `keymap.c`, the `enum combo_timing_buckets` defines your timing categories:

```c
enum combo_timing_buckets {
    TIMING_DEFAULT = 0,   // 50ms (or whatever COMBO_TERM is in config.h)
    TIMING_FAST = 1,      // 40ms - for quick-fire combos
    TIMING_SLOW = 2,      // 120ms - for deliberate combos
};
```

Each bucket needs a unique numeric ID (0, 1, 2, etc.).

### 2. Set Bucket Timing Values

Define the actual millisecond values for each bucket:

```c
static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 50,    // Default QMK combo term
    [TIMING_FAST] = 40,       // Faster detection
    [TIMING_SLOW] = 120,      // Slower detection
};
```

### 3. Map Combos to Buckets

Assign each combo to a timing bucket in `combo_timing_map[]`:

```c
static const uint8_t combo_timing_map[COMBO_COUNT] = {
    // Fast timing (40ms) - for quick combos
    [COMBO_TAB_FORWARD] = TIMING_FAST,
    [COMBO_TAB_BACKWARD] = TIMING_FAST,
    [COMBO_ESC] = TIMING_FAST,

    // Slow timing (120ms) - for deliberate combos
    [COMBO_ITERM] = TIMING_SLOW,
    [COMBO_COPY] = TIMING_SLOW,
    [COMBO_PASTE] = TIMING_SLOW,
    [COMBO_ALFRED] = TIMING_SLOW,

    // All others default to TIMING_DEFAULT (50ms)
};
```

**Note:** Any combo not listed in `combo_timing_map[]` will automatically use `TIMING_DEFAULT`.

## Examples

### Adding a New Timing Bucket

Say you want a "very fast" bucket for 30ms combos:

```c
enum combo_timing_buckets {
    TIMING_DEFAULT = 0,
    TIMING_FAST = 1,
    TIMING_VERY_FAST = 2,  // New bucket
    TIMING_SLOW = 3,       // Adjusted ID
};

static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 50,
    [TIMING_FAST] = 40,
    [TIMING_VERY_FAST] = 30,  // New timing value
    [TIMING_SLOW] = 120,
};

static const uint8_t combo_timing_map[COMBO_COUNT] = {
    [COMBO_ESC] = TIMING_VERY_FAST,  // Use new bucket
    [COMBO_TAB_FORWARD] = TIMING_FAST,
    [COMBO_COPY] = TIMING_SLOW,
};
```

### Assigning a Combo to a Different Bucket

To change `COMBO_REPEAT` from default to fast:

```c
static const uint8_t combo_timing_map[COMBO_COUNT] = {
    [COMBO_REPEAT] = TIMING_FAST,  // Add this line
    // ... other mappings
};
```

### Tuning Based on Feel

If a combo feels hard to trigger:
- **Decrease the time** (make it FASTER): 50ms → 40ms → 30ms
- **Increase latency**: shorter detection window, less false positives, but needs more precise timing

If a combo triggers too easily by accident:
- **Increase the time** (make it SLOWER): 50ms → 60ms → 80ms
- **Decrease sensitivity**: longer detection window, catches slower keypresses, higher false positive risk

**Recommended range:** 30-150ms. Most combos work well in the 40-100ms range.

## How It Works

1. When you press keys, QMK starts a timer
2. When all keys in a combo are detected within the timeout window, it fires
3. This implementation calls `get_combo_term()` which:
   - Checks if the combo has a mapping in `combo_timing_map[]`
   - Returns the corresponding timeout from `combo_timing_values[]`
   - Falls back to the default `COMBO_TERM` if not mapped

## Configuration Steps

### Step 1: Enable the Feature

In `config.h`, ensure you have:
```c
#define COMBO_TERM_PER_COMBO
```

### Step 2: Edit Timing Values

Open `keymap.c` and modify the `combo_timing_values[]` array to match your preferences.

### Step 3: Assign Combos

Update `combo_timing_map[]` to assign your combos to buckets.

### Step 4: Compile and Test

```bash
qmk compile -kb zsa/voyager -km rAxbQ
```

Try pressing the combos and adjust timings if needed.

## Testing Tips

1. **Test in a text editor** to see which combos feel sluggish or overly sensitive
2. **Adjust incrementally**: change by 10ms at a time, not 50ms
3. **Note which combos have slow keypresses**: these benefit from `TIMING_SLOW`
4. **Note which combos need quick detection**: these should use `TIMING_FAST`
5. **Recompile** after changes and flash your keyboard

## Common Patterns

### Aggressive/High-Sensitivity Setup
```c
static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 40,
    [TIMING_FAST] = 30,
    [TIMING_SLOW] = 80,
};
```
Good for fast typists who press keys nearly simultaneously.

### Conservative/Low-Sensitivity Setup
```c
static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 70,
    [TIMING_FAST] = 50,
    [TIMING_SLOW] = 150,
};
```
Good for preventing accidental combos and supporting slower keypresses.

### Balanced Setup (Current Default)
```c
static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 50,
    [TIMING_FAST] = 40,
    [TIMING_SLOW] = 120,
};
```
Good middle ground for most users.

## Troubleshooting

### "My combo never fires"
- Increase the timing value (e.g., 40ms → 60ms)
- Check that the combo keys are in `keymap.c` (line 116 onwards)
- Verify the combo is assigned correctly in the keymaps

### "My combo fires too easily"
- Decrease the timing value (e.g., 120ms → 100ms)
- Try moving the combo to `TIMING_FAST`
- Check that you're pressing the keys intentionally

### "Some combos work better with different timing"
- Create different buckets for different combo types
- Example: symbol combos might need 50ms, but navigation combos need 80ms

### "I want custom timing for a single combo"
- Create a new timing bucket specifically for it
- Or modify the existing bucket's value

## Reference: All Combos in rAxbQ

Combos defined in `keymap.c`:

| Combo | Keys | Output |
|-------|------|--------|
| COMBO_TAB_FORWARD | KC_S + KC_W | Ctrl+Tab |
| COMBO_TAB_BACKWARD | KC_T + KC_M | Ctrl+Shift+Tab |
| COMBO_ESC | KC_COMM + KC_DOT | Escape |
| COMBO_SK_ALT | KC_L + KC_D | Alt (one-shot) |
| COMBO_ITERM | LT(_NUMS, KC_P) + KC_H | Ctrl+\ |
| COMBO_SK_CTRL | KC_X + KC_M | Ctrl (one-shot) |
| COMBO_COPY | KC_X + KC_W | Cmd+C |
| COMBO_PASTE | KC_X + KC_M + KC_W | Cmd+V |
| COMBO_ALFRED | KC_Y + KC_H | Alt+Space |
| COMBO_REPEAT | KC_M + KC_W | Enter |
| *...and many more* | | |

See `enum custom_keycodes` (line 17) and `enum combos` (line 63) for the complete list.

## Advanced: Creating a Modifier-Specific Bucket

For combos with multiple modifier keys, you might want slower detection:

```c
enum combo_timing_buckets {
    TIMING_DEFAULT = 0,
    TIMING_FAST = 1,
    TIMING_MOD_COMBOS = 2,  // For modifier combos
    TIMING_SLOW = 3,
};

static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 50,
    [TIMING_FAST] = 40,
    [TIMING_MOD_COMBOS] = 100,  // Longer window for modifier combos
    [TIMING_SLOW] = 120,
};

static const uint8_t combo_timing_map[COMBO_COUNT] = {
    [COMBO_SK_ALT] = TIMING_MOD_COMBOS,
    [COMBO_SK_CTRL] = TIMING_MOD_COMBOS,
    [COMBO_SK_LGUI] = TIMING_MOD_COMBOS,
    // ... other mappings
};
```

## Need Help?

- Check the QMK combo documentation: https://docs.qmk.fm/features/combo
- Review `process_combo.h` in the QMK firmware for technical details
- Test individual combos by pressing the keys slowly first, then faster

Happy combo tuning!
