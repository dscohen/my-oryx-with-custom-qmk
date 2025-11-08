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
    TIMING_VERTICAL = 3,  // 80ms - for vertical combos (same column, adjacent rows)
};
```

Each bucket needs a unique numeric ID (0, 1, 2, 3, etc.).

### 2. Set Bucket Timing Values

Define the actual millisecond values for each bucket:

```c
static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 50,     // Default QMK combo term
    [TIMING_FAST] = 40,        // Faster detection for quick combos
    [TIMING_SLOW] = 120,       // Slower detection for deliberate combos
    [TIMING_VERTICAL] = 80,    // Balanced timing for vertical combos
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

## Vertical Combos Explained

Vertical combos are key pairs where both keys are in the **same column** but **different rows** (stacked vertically). They're physically easier to press simultaneously because your fingers move naturally up and down.

### Why Vertical Combos Need Special Timing

**Vertical combos have unique timing characteristics:**
- **Easier to trigger:** Your fingers naturally move in the same column, making them easier to press together
- **Higher false positive risk:** If you're typing adjacent keys in the same column quickly, you might accidentally trigger a combo
- **Benefits from longer window:** A slightly longer timeout (80ms) helps catch intentional presses while filtering out accidental ones

### Vertical Combos in rAxbQ

**Left side vertical combos:**
- `COMBO_SK_LGUI`: KC_D ↔ KC_T (column 3)
- `COMBO_TAB_BACKWARD`: KC_T ↔ KC_M (column 3)
- `COMBO_QSTN`: KC_L ↔ KC_R (column 2)
- `COMBO_EXCLM`: KC_R ↔ KC_X (column 2)
- `COMBO_MEH_LAYER` / `COMBO_MEH`: KC_C ↔ KC_S (column 4)
- `COMBO_TAB_FORWARD`: KC_S ↔ KC_W (column 4)
- `COMBO_SEMICOLON`: KC_V ↔ KC_G (column 5)
- `COMBO_PIPE`: KC_G ↔ KC_J (column 5)

**Right side vertical combos:**
- `COMBO_SK_RGUI`: KC_O ↔ KC_A (column 2)
- `COMBO_COLON`: KC_A ↔ KC_COMM (column 2)
- `COMBO_ALFRED`: KC_Y ↔ KC_H (column 1)
- `COMBO_FSLASH`: KC_H ↔ KC_F (column 1)
- `COMBO_HASH`: KC_U ↔ KC_E (column 3)
- `COMBO_PERCENT`: KC_E ↔ KC_DOT (column 3)

All vertical combos are pre-configured with `TIMING_VERTICAL` (80ms) for optimal detection.

## Examples

### Adjusting Vertical Combo Timing

If your vertical combos feel sluggish, reduce the timing:

```c
static const uint16_t combo_timing_values[] = {
    [TIMING_VERTICAL] = 60,  // Faster - from 80ms to 60ms
};
```

If they trigger too easily (false positives), increase it:

```c
static const uint16_t combo_timing_values[] = {
    [TIMING_VERTICAL] = 100,  // Slower - from 80ms to 100ms
};
```

### Adding a New Timing Bucket

Say you want a "very fast" bucket for 30ms combos:

```c
enum combo_timing_buckets {
    TIMING_DEFAULT = 0,
    TIMING_FAST = 1,
    TIMING_SLOW = 2,
    TIMING_VERTICAL = 3,
    TIMING_VERY_FAST = 4,     // New bucket
};

static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 50,
    [TIMING_FAST] = 40,
    [TIMING_SLOW] = 120,
    [TIMING_VERTICAL] = 80,
    [TIMING_VERY_FAST] = 30,  // New timing value
};

static const uint8_t combo_timing_map[COMBO_COUNT] = {
    [COMBO_ESC] = TIMING_VERY_FAST,  // Use new bucket
    [COMBO_TAB_FORWARD] = TIMING_VERTICAL,
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
    [TIMING_VERTICAL] = 60,  // Faster vertical detection
};
```
Good for fast typists who press keys nearly simultaneously. Vertical combos fire quickly to match typing speed.

### Conservative/Low-Sensitivity Setup
```c
static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 70,
    [TIMING_FAST] = 50,
    [TIMING_SLOW] = 150,
    [TIMING_VERTICAL] = 100,  // Slower vertical detection
};
```
Good for preventing accidental combos and supporting slower keypresses. Vertical combos have a longer window to prevent false positives.

### Balanced Setup (Current Default)
```c
static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 50,
    [TIMING_FAST] = 40,
    [TIMING_SLOW] = 120,
    [TIMING_VERTICAL] = 80,   // Balanced vertical detection
};
```
Good middle ground for most users. Vertical combos are neither too quick nor too slow.

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

## Advanced: Fine-Tuning Vertical Combos by Column

Different columns on your keyboard might benefit from different vertical timings. For example, you might want faster detection on one side than the other:

```c
enum combo_timing_buckets {
    TIMING_DEFAULT = 0,
    TIMING_FAST = 1,
    TIMING_SLOW = 2,
    TIMING_VERTICAL = 3,
    TIMING_VERTICAL_LEFT = 4,   // Separate timing for left-side verticals
    TIMING_VERTICAL_RIGHT = 5,  // Separate timing for right-side verticals
};

static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 50,
    [TIMING_FAST] = 40,
    [TIMING_SLOW] = 120,
    [TIMING_VERTICAL] = 80,
    [TIMING_VERTICAL_LEFT] = 70,   // Faster for left side
    [TIMING_VERTICAL_RIGHT] = 90,  // Slower for right side
};

static const uint8_t combo_timing_map[COMBO_COUNT] = {
    // Left-side verticals
    [COMBO_SK_LGUI] = TIMING_VERTICAL_LEFT,
    [COMBO_TAB_BACKWARD] = TIMING_VERTICAL_LEFT,
    [COMBO_QSTN] = TIMING_VERTICAL_LEFT,
    [COMBO_EXCLM] = TIMING_VERTICAL_LEFT,

    // Right-side verticals
    [COMBO_SK_RGUI] = TIMING_VERTICAL_RIGHT,
    [COMBO_COLON] = TIMING_VERTICAL_RIGHT,
    [COMBO_ALFRED] = TIMING_VERTICAL_RIGHT,
    [COMBO_FSLASH] = TIMING_VERTICAL_RIGHT,
    // ... more mappings
};
```

This is useful if you notice one side of your keyboard naturally types faster than the other.

## Advanced: Creating a Modifier-Specific Bucket

For combos with multiple modifier keys, you might want slower detection:

```c
enum combo_timing_buckets {
    TIMING_DEFAULT = 0,
    TIMING_FAST = 1,
    TIMING_VERTICAL = 2,
    TIMING_MOD_COMBOS = 3,  // For modifier combos
    TIMING_SLOW = 4,
};

static const uint16_t combo_timing_values[] = {
    [TIMING_DEFAULT] = 50,
    [TIMING_FAST] = 40,
    [TIMING_VERTICAL] = 80,
    [TIMING_MOD_COMBOS] = 100,  // Longer window for modifier combos
    [TIMING_SLOW] = 120,
};

static const uint8_t combo_timing_map[COMBO_COUNT] = {
    [COMBO_SK_ALT] = TIMING_MOD_COMBOS,
    [COMBO_SK_CTRL] = TIMING_MOD_COMBOS,
    [COMBO_SK_LGUI] = TIMING_MOD_COMBOS,
    [COMBO_SK_RGUI] = TIMING_MOD_COMBOS,
    // ... other mappings
};
```

Note: `SK_LGUI` and `SK_RGUI` are both vertical combos, so you could also use `TIMING_VERTICAL` for them if that works better.

## Need Help?

- Check the QMK combo documentation: https://docs.qmk.fm/features/combo
- Review `process_combo.h` in the QMK firmware for technical details
- Test individual combos by pressing the keys slowly first, then faster

Happy combo tuning!
