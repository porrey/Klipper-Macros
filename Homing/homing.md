# Homing User Guide (Klipper Macros)

A set of macros for safe, controlled homing of the X, Y, and Z axes. This guide documents the macros and configuration files:

- `homing_parameters` (behavior and default configuration variables — in `homing-config.cfg`)
- `G27` (park the toolhead — in `G27.cfg`)
- `G28` (auto home — in `G28.cfg`)
- `G29` (bed mesh calibration — in `G29.cfg`)
- `HOMING_STATUS` / `HOMING_STATUS_X` / `HOMING_STATUS_Y` / `HOMING_STATUS_Z` (report homing state)
- `HOME_X` / `HOME_Y` / `HOME_Z` (individually home each axis)
- `SAFE_XY` (move to the safe XY position)
- `SAFE_XYZ` (park the toolhead at the safe XY/Z position)

---

## Overview

### What this does

These macros replace the built-in `G28` with a safer, stepwise homing sequence:

1. Raises Z (or force-moves if Z is not yet homed) to prevent bed collisions.
2. Homes each axis individually in order: X → Y → Z.
3. Before homing Z, moves X/Y to the configured safe position (`SAFE_XY`).
4. After homing, loads the default bed mesh (if Z was homed).

### What you need

- Correct **homing parameters** in `homing_parameters` (`homing-config.cfg`).
- `force_move` enabled in your `printer.cfg` (required when Z is not yet homed):
  ```ini
  [force_move]
  enable_force_move: True
  ```

---

## Installation

1. Copy all files from the `Homing` folder into a folder called `Homing` on your printer (e.g. alongside your `printer.cfg`):
   - `homing.cfg`
   - `homing-config.cfg`
   - `G27.cfg`
   - `G28.cfg`
   - `G29.cfg`

2. Include `homing.cfg` from your `printer.cfg`:

   ```ini
   [include Homing/homing.cfg]
   ```

3. Edit `Homing/homing-config.cfg` to match your printer's geometry.
4. Restart Klipper.

---

## Configuration

### `homing_parameters` (`homing-config.cfg`)

This macro stores all homing geometry as variables. It does **not** execute motion.

| Variable | Default | Meaning |
|---|---:|---|
| `variable_safe_x` | `100` | Safe X position for the nozzle when parking or homing Z |
| `variable_safe_y` | `110` | Safe Y position for the nozzle when parking or homing Z |
| `variable_x_hop` | `5` | Distance to move X away from the endstop before/after homing |
| `variable_y_hop` | `5` | Distance to move Y away from the endstop before/after homing |
| `variable_feed_rate` | `1500` | Feed rate (mm/min) used for all homing moves |
| `variable_safe_z` | `10` | Z height (mm) used for safe travel moves |

---

## Command Reference

### `G28` — Auto Home

Homes one or more axes. Replaces the built-in `G28` (original is renamed to `G28.1`).

#### Parameters

| Parameter | Description |
|---|---|
| `X` | Home X axis |
| `Y` | Home Y axis |
| `Z` | Home Z axis |

If no axes are specified, all three axes are homed (X → Y → Z).

After homing Z, the default bed mesh profile is loaded automatically.

#### Example

```gcode
G28        ; home all axes
G28 X      ; home X only
G28 X Y    ; home X and Y
```

---

### `HOME_X` — Home X Axis

Safely homes the X axis.

#### What it does

1. If Z is homed and below `variable_safe_z`, raises Z to `variable_safe_z`.
2. If Z is not homed, force-moves Z up by `variable_safe_z`.
3. If X is homed and within `variable_x_hop`, moves X away from the endstop.
4. If X is not homed, force-moves X by `variable_x_hop`.
5. Runs `G28.1 X` to home X.
6. Moves X away from the endstop by `variable_x_hop`.

#### Usage

```gcode
HOME_X
```

---

### `HOME_Y` — Home Y Axis

Safely homes the Y axis. Requires X to be homed first.

#### What it does

1. If X is not homed, displays an informational message and exits.
2. If Y is homed and within `variable_y_hop`, moves Y away from the endstop.
3. If Y is not homed, force-moves Y by `variable_y_hop`.
4. Runs `G28.1 Y` to home Y.
5. Moves Y away from the endstop by `variable_y_hop`.

#### Usage

```gcode
HOME_Y
```

---

### `HOME_Z` — Home Z Axis

Safely homes the Z axis. Requires X and Y to be homed first.

#### What it does

1. If X or Y is not homed, displays an informational message and exits.
2. Moves X/Y to the safe homing position (`SAFE_XY`).
3. Runs `G28.1 Z` to home Z.
4. Moves Z to `variable_safe_z`.

#### Usage

```gcode
HOME_Z
```

---

### `SAFE_XY` — Move to Safe XY Position

Moves X/Y to the safe homing position (`variable_safe_x`, `variable_safe_y`). Requires X and Y to be homed.

#### Usage

```gcode
SAFE_XY
```

---

### `SAFE_XYZ` — Park Toolhead

Moves X/Y/Z to the safe parked position. Requires all axes to be homed.

#### Usage

```gcode
SAFE_XYZ
```

---

### `G27` — Park Toolhead

Calls `SAFE_XYZ` to park the toolhead at the configured safe position.

#### Usage

```gcode
G27
```

---

### `G29` — Bed Mesh Calibration

Runs a full bed mesh calibration sequence:

1. Sets starting acceleration (`M204 S5000`).
2. Turns off the hotend and fan.
3. Homes all axes (`G28`).
4. Runs `BED_MESH_CALIBRATE`.
5. Turns off the bed heater.
6. Saves the configuration (`SAVE_CONFIG`).
7. Parks the toolhead (`G27`).

#### Usage

```gcode
G29
```

---

### `HOMING_STATUS` — Display Homing Status

Reports whether X, Y, and Z are homed.

#### Usage

```gcode
HOMING_STATUS
```

#### Output example

```
Homing status: X=YES  Y=YES  Z=NO (homed_axes=xy)
```

---

### `HOMING_STATUS_X` / `HOMING_STATUS_Y` / `HOMING_STATUS_Z`

Reports whether the individual axis is homed.

#### Usage

```gcode
HOMING_STATUS_X
HOMING_STATUS_Y
HOMING_STATUS_Z
```

---

## Safety notes

- `force_move` must be enabled in `printer.cfg` so that `HOME_X` and `HOME_Y` can raise Z before homing when Z is not yet homed.
- Set `variable_safe_x` and `variable_safe_y` to a position above the center of the bed (or your probe point) so that `HOME_Z` homes Z at a safe location.
- Set `variable_safe_z` high enough to clear any bed hardware (clips, magnets, etc.) during XY travel.

---

## License / Attribution

Written by **Daniel Porrey**.
Feel free to include this guide in your GitHub repository alongside the macros.
