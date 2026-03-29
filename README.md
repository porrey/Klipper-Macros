# Klipper-Macros
My Klipper Macros

1. **Homing** — A set of macros for safe, controlled homing of the X, Y, and Z axes. Replaces the built-in `G28` with a stepwise sequence that raises Z before homing X/Y, moves to a configurable safe XY position before homing Z, and loads the default bed mesh after homing. Also provides `G27` (park toolhead), `G29` (bed mesh calibration), and individual `HOME_X` / `HOME_Y` / `HOME_Z` macros with status reporting. Install by dropping all files into a `Homing` folder and adding `[include Homing/homing.cfg]` to `printer.cfg`.

   [Full documentation →](https://github.com/porrey/Klipper-Macros/blob/main/Homing/homing.md)

1. **Nozzle Clean** — Wipe the nozzle on a cleaning pad (silicone brush, brass brush, scrub pad, etc.) to remove ooze and debris before or after a print. Supports straight-line and zig-zag wipe patterns, automatic wipe-height calculation from pad geometry, filament retraction, configurable temperatures and pass counts, and a dry-run test mode. Install by dropping all files into a `Nozzle-Clean` folder and adding `[include Nozzle-Clean/nozzle-clean.cfg]` to `printer.cfg`.

   [Full documentation →](https://github.com/porrey/Klipper-Macros/blob/main/Nozzle-Clean/nozzle-clean.md)
