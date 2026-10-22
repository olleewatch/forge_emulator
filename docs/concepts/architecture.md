# Architecture

## The big picture

```
 your-face.js  ──►  watch firmware  ──►  Espruino JS engine
                    (Forge interpreter)        │
                         ▲                     ▼
   buttons, gestures,    │              Ollee.* calls
   BLE, clock events ────┘              (display, beep, LED, sensors, BLE)
```

A Forge watchface is a single JavaScript file. The watch firmware stores it,
runs it on an embedded [Espruino](https://www.espruino.com/) engine, and calls
functions in it by name when things happen: the face comes on screen, a button
is pressed, a minute passes. Your script reacts by calling the `Ollee` API.

The emulator in this repo copies that setup in the browser. `js/ollee.js`
provides the same `Ollee` object, and `js/app.js` calls your hooks the way the
firmware does.

## Script slots

- **The watch has three Forge slots**, so up to three Forge faces can be on it
  at once. By convention, Forge face names start with the letter `F`, followed
  by a letter for the slot: `FA`, `FB` and `FC`. The name shows on the weekday
  digits while the face is on screen, which tells it apart from the watch's
  built-in faces.
- The emulator runs one face at a time, in slot `FA`.
- **8158 bytes maximum** per script (UTF-8 encoded). The emulator's size bar shows how
  close you are, and the log warns when a script is over.
- **The script stays loaded** while you're on other faces. Its variables keep
  their values, and background subscriptions keep running (see
  [Lifecycle](lifecycle.md)).
- `init()` runs when the script is uploaded and each time the watch boots.

## What a face can do

- Draw on the 6 clock digits and 2 day digits, and switch on the colon, LAP, PM
  and HR indicators ([Display](display.md)).
- React to the A and L buttons (short press, long press, raw down/up), wrist
  tilt and triple tap ([Input](input.md)).
- Run up to 4 timers at once.
- Beep, and turn the LED on and off.
- Read the temperature, step count, battery voltage, a hardware random number,
  and the location the paired phone set.
- Talk to a phone app over Bluetooth LE.
- Wake on each new minute or hour, even while another face is showing.

## What a face can't do

- Use the M button. It always switches between your face and the home screen.
- Draw on the weekday digits, or set the bell (alarm) or chime indicators.
  Those belong to the system.
- Keep running after an uncaught error. The face is disabled until the script
  is reloaded.
- Use modules, `require`, the network, or storage. It's one self-contained file.

## Emulator vs. watch

The emulator mirrors the firmware closely, but some things differ:

- It runs your browser's JavaScript engine, which supports more of the
  language than Espruino. Test on the watch before you share a face.
- Espruino doesn't hoist function declarations: a function only exists once
  the script has run past it. Code at the top level that calls a function
  declared further down works in the emulator but fails on the watch.
  Calls from hooks are fine, because hooks run after the whole script has loaded.
- Sensors are simulated: steps are fixed at 5000, temperature drifts around
  23 °C, battery reads 2.9 V, and location is fixed to Toronto.
- Timers are as accurate as the browser's, and speed or memory limits aren't
  emulated.
