# AGENTS.md

Orientation for AI coding agents working in this repository.

This repo is the **Ollee Watch Forge emulator**: a static browser page that runs
Forge scripts (small JavaScript watchfaces) against a mock-up of the Ollee
Watch's segment LCD. You'll most likely be asked to write or fix a watchface
script: read the rules below and [docs/api/README.md](docs/api/README.md).

## Writing a watchface: hard rules

A watchface is one plain `.js` file. It is not a module, and it has no imports
and no build step. The watch runs it on Espruino; the emulator runs it in the
browser.

- **Hooks are found by their exact top-level name.** Declare them as top-level
  `function` declarations (or top-level `var`/`const`): `init`, `enter`,
  `exit`, `a_short`, `a_long`, `l_short`, `l_long`, `tilt`, `tap`, `button`,
  `ble_rx`, `clock_minute`, `clock_hour`. Never wrap them in an object, IIFE, or
  class.
- **Event hooks only fire if their bit is in `subscribe`**, for example
  `const subscribe = E_A_SHORT + E_L_SHORT;`. `init`, `enter` and `exit` don't
  need a bit.
- **The M button isn't yours.** It switches between the face and the home
  screen and calls `exit()`/`enter()`. There's no M hook.
- **Draw in `enter()`, not `init()`.** `init()` runs at upload/boot, before the
  face is on screen, and draw calls are ignored then (and whenever
  `Ollee.active()` is false).
- **Timers:** use `Ollee.setTimeout`/`Ollee.setInterval`, not the globals.
  They share **4 slots** and return `-1` when they're all used. Leaving the face
  cancels them, so restart them in `enter()` and reset your stored ids in
  `exit()`.
- **An uncaught error disables the face** on the watch. Wrap risky code in
  `try`/`catch`.
- **Size limit: 8158 bytes** (UTF-8). Keep scripts small. Minification never
  renames identifiers, so names cost bytes.
- **Display:** `Ollee.drawMain(text)` shows up to 6 characters on the clock
  digits, right-aligned. `Ollee.drawDay(text)` shows 2 on the day digits. It's a
  7-segment LCD, so many letters only approximate. Check them with the
  emulator's **Character set** button. The weekday digits show the face's
  slot name (`FA`, `FB` or `FC`) and scripts can't draw there.
- **`Date` has no timezone.** `new Date()` gives the watch's wall-clock time
  directly; `getTimezoneOffset()` is 0.
- Use ES5-style code (`var`, `function`) when in doubt. Espruino doesn't support
  every modern JS feature, and the emulator won't catch that.
- **Declare functions before top-level code calls them.** Espruino doesn't hoist
  function declarations, so `var t = make();` above `function make() {...}`
  fails on the watch. The emulator's browser engine hoists, so it won't catch it.

Start from [examples/template.js](examples/template.js), or a closer example in
[examples/](examples/).

## Testing a script

There's no headless test runner. A human (or a browser-automation tool) runs
the script in the emulator:

1. Open `index.html` in a browser.
2. Paste the script into the editor (or **Load file**) and click **Run**.
3. Press the on-screen buttons, or the keys `l` `m` `a` `t` (tilt) `p` (tap).
   Hold several keys at once for combos.
4. Read the **log** panel. **Copy log** puts it on the clipboard so it can be
   handed back to you.

The Bluetooth panel sends BLE writes to `ble_rx`, and the Clock panel fires
`clock_minute`/`clock_hour` on demand.

## Map of the docs

- [docs/concepts/lifecycle.md](docs/concepts/lifecycle.md): when each hook runs
- [docs/concepts/display.md](docs/concepts/display.md): what the LCD can show
- [docs/concepts/input.md](docs/concepts/input.md): buttons, gestures, BLE, clock events
- [docs/concepts/power.md](docs/concepts/power.md): writing faces that don't drain the battery
- [docs/api/README.md](docs/api/README.md): every `Ollee.*` call and constant
