# NOZZLE_CLEAN User Guide (Klipper Macro)

Clean your nozzle by wiping it on a defined “pad” area on the bed (silicone brush, brass brush, scrub pad, etc.). This guide documents the macros:

- `_NOZZLE_CLEAN_CONFIG` (configuration variables only)
- `NOZZLE_CLEAN` (run the cleaning routine)
- `NOZZLE_CLEAN_TEST` (safe dry-run: no heat, stays above pad)
- `NOZZLE_PAD_EDGE` (trace pad perimeter to validate geometry)

---

## Overview

### What this does
`NOZZLE_CLEAN`:
1. Optionally starts heating the nozzle (non-blocking).
2. Homes the printer if needed.
3. Moves above the cleaning pad area.
4. Waits for target temperature (if needed).
5. Drops to a wipe height.
6. Wipes back and forth for a number of passes.
7. Lifts to a safe Z and optionally turns the hotend off.
8. Restores prior gcode state.

### What you need
- A defined “pad” location on your bed where it is safe to wipe.
- Correct **pad geometry** values in `_NOZZLE_CLEAN_CONFIG`.
- A safe **wipe Z height** that contacts the pad without crashing.

---

## Installation

1. Copy the macro block into a Klipper config file (e.g. `macros_nozzle_clean.cfg`).
2. Include it from `printer.cfg`:

   ```ini
   [include macros_nozzle_clean.cfg]
   ```

3. Restart Klipper.

---

## Quick Start

### Basic use
```gcode
NOZZLE_CLEAN
```

### Set a custom temperature and number of passes
```gcode id="27r0mg"
NOZZLE_CLEAN TEMP=210 PASSES=8
```

### Override wipe height for a single run
```gcode id="creavt"
NOZZLE_CLEAN Z=0.6
```

### Keep the hotend on when complete
```gcode id="mnzeqm"
NOZZLE_CLEAN KEEP_HOTEND_ON=1
```

### Turn the hotend off when complete
```gcode id="p32y1v"
NOZZLE_CLEAN KEEP_HOTEND_ON=0
```

---

## Configuration: `_NOZZLE_CLEAN_CONFIG`

This macro stores all settings as variables. It does **not** execute motion.

Edit these values to match your machine and pad location.

### Pad geometry (mm)
| Variable | Meaning |
|---|---|
| `variable_pad_x` | Pad “start” X (one end of wipe line when wiping in X) |
| `variable_pad_y` | Pad Y (for X wipes) **or** pad start Y (for Y wipes) |
| `variable_pad_width` | Pad size in X (left-to-right) |
| `variable_pad_depth` | Pad size in Y (front-to-back) |
| `variable_pad_z` | Default wipe height (absolute Z) |

> Tip: `pad_x`, `pad_y` are treated like the **minimum corner** (front/left depending on your coordinate system). Width/depth extend positively.

### Behavior
| Variable | Meaning |
|---|---|
| `variable_wipe_axis` | `"X"` wipes back/forth in **X** at constant Y, `"Y"` wipes back/forth in **Y** at constant X |
| `variable_safe_z` | Travel Z height (absolute) used before/after wiping |
| `variable_travel_f` | Travel feedrate (mm/min) |
| `variable_wipe_f` | Wipe feedrate (mm/min) |

### Defaults
| Variable | Meaning |
|---|---|
| `variable_default_temp` | Default cleaning temperature if `TEMP` not provided |
| `variable_default_passes` | Default number of wipe passes |
| `variable_default_keep_hotend_on` | `1` keep hotend on, `0` turn hotend off |
| `variable_always_heat` | `True` always heat/wait to `TEMP`; `False` only heat/wait if below `(TEMP-2)` |

---

## Command Reference

## `NOZZLE_CLEAN`

Clean the nozzle using the configured pad.

### Parameters
| Parameter | Type | Default | Description |
|---|---:|---:|---|
| `TEMP` | int | `default_temp` | Nozzle cleaning temperature in °C |
| `PASSES` | int | `default_passes` | Number of wipe passes |
| `Z` | float | `pad_z` | Override wipe height (absolute Z) for this run |
| `KEEP_HOTEND_ON` | int (0/1) | `default_keep_hotend_on` | Keep nozzle hot afterward (1) or turn off (0) |

### Behavior details
- **State safety:** Uses `SAVE_GCODE_STATE` / `RESTORE_GCODE_STATE` to preserve modes/settings.
- **Homing:** If `printer.toolhead.homed_axes != "xyz"`, it runs `G28`.
- **Positioning:** Uses `G90` (absolute moves).
- **Heating logic:**
  - If `always_heat=True`: always `M104` then always `M109`.
  - If `always_heat=False`: heat/wait only if current temp is below `(TEMP - 2)`.

### Example
```gcode id="m4dxa5"
NOZZLE_CLEAN TEMP=205 PASSES=6 Z=0.8 KEEP_HOTEND_ON=1
```

---

## `NOZZLE_CLEAN_TEST`

Dry-run of `NOZZLE_CLEAN` without touching the pad and without heating.

### What it does
- Calls `NOZZLE_CLEAN TEMP=0` and raises Z by ~3mm above configured wipe height.

### Usage
```gcode id="nv57ub"
NOZZLE_CLEAN_TEST
```

### Optional
```gcode id="spk69l"
NOZZLE_CLEAN_TEST PASSES=10
```

> Note: This is intended to validate motion/pathing safely. Still ensure the pad area is clear.

---

## `NOZZLE_PAD_EDGE`

Traces the pad perimeter (optionally inset) to validate pad geometry and positioning.

### Parameters
| Parameter | Type | Default | Description |
|---|---:|---:|---|
| `PASSES` | int | `2` | Number of perimeter loops |
| `INSET` | float | `0.0` | Inset from pad edges (mm) |
| `Z` | float | `5.0` | Z height to trace at (absolute) |

### Usage
Trace exact pad perimeter:
```gcode id="1dypgx"
NOZZLE_PAD_EDGE
```

Trace an inset perimeter:
```gcode id="s02oef"
NOZZLE_PAD_EDGE PASSES=3 INSET=2
```

Lower it to contact the pad (be careful):
```gcode id="gfulgz"
NOZZLE_PAD_EDGE PASSES=2 INSET=1 Z=0.6
```

### Validation checks
`NOZZLE_PAD_EDGE` will error if:
- `PASSES < 1`
- `INSET < 0`
- `INSET` is too large for the pad size (`(w - 2*inset) <= 0` or `(d - 2*inset) <= 0`)

---

## How pad targeting works

The macro computes the **center line** on the non-wipe axis:

- `cx = pad_x + (pad_width / 2)`
- `cy = pad_y + (pad_depth / 2)`

Then:
- If wiping in **X** (`wipe_axis="X"`): it moves to `X=pad_x`, `Y=cy` and wipes between `pad_x` and `pad_x + pad_width`.
- If wiping in **Y** (`wipe_axis="Y"`): it moves to `X=cx`, `Y=pad_y` and wipes between `pad_y` and `pad_y + pad_depth`.

---

## Recommended setup workflow

1. **Start with safe Z testing**
   - Run:
     ```gcode
     NOZZLE_PAD_EDGE Z=10
     ```
   - Confirm the toolhead traces the correct rectangle above the pad.

2. **Dial in pad geometry**
   - Adjust `pad_x`, `pad_y`, `pad_width`, `pad_depth` until the trace aligns.

3. **Find a safe wipe height**
   - Start higher (e.g. `Z=2.0`) and step down carefully.
   - Once known, set `variable_pad_z` to your preferred default.

4. **Run a real cleaning**
   - Example:
     ```gcode
     NOZZLE_CLEAN TEMP=200 PASSES=5 KEEP_HOTEND_ON=1
     ```

---

## Safety notes

- Verify the pad area is clear and the nozzle won’t collide with clips, magnets, bed screws, or printed parts.
- Use conservative speeds and higher Z while testing.
- If your cleaning pad is abrasive (e.g., brass brush), ensure your wipe height won’t bend mounts or overload the toolhead.

---

## Troubleshooting

### “It wipes in the wrong direction”
- Set `variable_wipe_axis` to `"X"` or `"Y"` correctly.

### “It doesn’t heat sometimes”
- If `variable_always_heat` is `False`, the macro only heats/waits when below `(TEMP - 2)`.
- Set `variable_always_heat: True` if you want consistent heating behavior.

### “It’s not wiping on the pad”
- Use `NOZZLE_PAD_EDGE` to validate geometry and location.
- Re-check `pad_x`, `pad_y`, `pad_width`, `pad_depth`.
- Confirm your coordinate system and bed origin match your assumptions.

### “It’s too aggressive / too slow”
- Adjust:
  - `variable_wipe_f` (wipe speed)
  - `variable_travel_f` (travel speed)
  - `PASSES` (number of wipes)

---

## Changelog

- **v1.0.0** (Last Updated: 2026-02-27)
  - Initial release.

---

## License / Attribution

Written by **Daniel Porrey**.
Feel free to include this guide in your GitHub repository alongside the macro.
```