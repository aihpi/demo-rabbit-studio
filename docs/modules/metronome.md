# Metronome

## Sources

- Original: `CP5_TT_20251119_Metronoom5.html` (159 lines, Dutch labels, "CP5" neon look, no external resources).
- Static analysis: `report_CP5_TT_20251119_Metronoom5.json`. It is the only report for this module.
- Target files: `01_frontend/instruments/metronome/index.js`. Its styles are rules in `styles/components.css`; there is no CSS file of its own. It uses the shared modules `ui/knob.js`, `ui/led.js`, `ui/lcd.js` and `ui/pad.js`. It does not need `audio/noise.js`.

## Purpose

The Metronome gives one short oscillator click on every quarter note of the global transport. It keeps the small "Slot Machine" random-number widget as a second plate in the same tab. The click no longer has its own timer or tempo; it follows the header transport.

## Feature parity

| Feature in the source | Source | In RABBIT Studio |
|---|---|---|
| Start/Stop toggle with label flip "Start"/"Stop"; Stop only calls `clearInterval` | `#startStop` l.88, l.126-142 | Moves to the global header (start, stop). The instrument has an on/off switch instead, stored in the project as `on.metronome`. Switching off calls `onStop()`; switching on while the transport runs calls `onStart(time)`. `onStop()` cancels scheduled clicks and ends a sounding click with a short fade (the original Stop did not). |
| First click sounds one full beat after pressing Start, because `setInterval` does not fire at once | l.132-136 | Replaced. The first click sounds on step 0 at the transport start time, because the lookahead scheduler calls `onStep(0, ...)` at once. The delay was a side effect of `setInterval`, not a musical choice. |
| Click sound: new oscillator, lowpass filter and gain per click; waveform sine, sawtooth or square; one fixed pitch; stopped after `noteLength` ms | `playClick` l.104-124 | Kept. Same chain per click. Start and stop use `time` and `time + noteLength/1000` on the audio clock instead of `osc.start()` and `setTimeout`. |
| Pitch is `220 * 2^(octave-4)`: octave 2 = 55 Hz, 3 = 110, 4 = 220, 5 = 440, 6 = 880. Default is octave 2 (first option). No note choice, accent or bar length | l.111, l.79-85 | Kept, including the default of 55 Hz. No accent or note choice is added. |
| Beat indicator: 50 px circle, magenta "active" class on each tick, cleared by `setTimeout` after 100 ms; driven by the timer, not by audio time | `#indicator` l.89, l.134-135 | Kept as a large LED (`ui/led.js`) in red. It lights for 100 ms at the click time. `onStep` writes `{ stepIndex, time }` into a ring buffer, and the mounted interface reads it in the shared animation loop, so light and sound agree and `onStep` does audio only. It is not drawn while the tab is hidden. |
| Slot Machine: Min (default 1), Max (default 100), display "---", Spin button. Spin writes a random integer every 100 ms and stops after 21 writes (about 2.1 s). The last value stays. No payout, reels or sound | l.91-97, l.144-156 | Kept as a widget with the same timing. It is pure UI and does not follow the transport. The display becomes an LCD (`ui/lcd.js`), Spin becomes a pad (`ui/pad.js`). |
| "Nothing else": no recording, export, presets or keyboard shortcuts | report features (the file ends at l.159) | Nothing added. Project save and load comes from the app, not from this module. |
| Tempo (BPM) input, 40 to 240, default 120; read only at Start; typed values not validated | `#tempo` l.68, l.130 | Moves to the global header (tempo). The range is 40 to 300 BPM with a default of 120, set in `docs/plan.md`, section Transport. Tempo changes now act at once (see Transport behaviour). |
| Waveform select: sine (default), sawtooth, square; read on every click | `#waveform` l.70-74, l.109 | Kept as a three-position selector. Read per click from state. |
| Filter frequency range 100 to 10000 Hz, default 1000, step 1; lowpass cutoff read per click | `#filter` l.76, l.114 | Kept as a knob (linear scale, see Controls). |
| Nootlengte (ms), range 1 to 40, default 10; osc stop delay | `#noteLength` l.77, l.123 | Kept as a knob, label "Note length". |
| Octaaf select 2 to 6, default 2 | `#octave` l.79-85 | Kept as a five-position selector, label "Octave". |
| Volume range 0 to 1, step 0.01, default 0.5; set as `gain.value` | `#volume` l.87, l.116 | Moves to the Console strip (volume). The click gain stays fixed at 0.5 inside the voice. The strip default gain is owned by `docs/modules/shell.md` and `docs/modules/console.md`. |
| Start/Stop button | `#startStop` l.88 | Moves to the global header. Removed from the tab. |
| Min and Max number inputs (defaults 1 and 100, no limits) | `#minVal`, `#maxVal` l.93-94 | Kept as two number fields. Min greater than Max is swapped at spin time (see Defects fixed). |
| Spin button, never disabled during a spin | `#spin` l.96 | Kept, but disabled while a spin runs (see Defects fixed). |
| Wall-clock start at multiples of 4 seconds and its countdown | Not in this file (report: clockSync "None") | Removed by decision. Nothing to remove here, the file never had it. |

Where the source and the target differ in timing, the target wins: the lookahead scheduler replaces `setInterval`, because the global transport owns time for all instruments. The original has one tempo source (its own input); the header tempo is now the only one. The transport has one late-step guard (see `docs/plan.md`, section Transport); this module has none.

## Controls

| Control | Component type | Range | Default | Source |
|---|---|---|---|---|
| Waveform | Three-position selector | sine, sawtooth, square | sine | l.70-74 |
| Filter | Knob (`ui/knob.js`), Hz | 100 to 10000, step 1 | 1000 | l.76 |
| Note length | Knob, ms | 1 to 40, step 1 | 10 | l.77 |
| Octave | Five-position selector | 2, 3, 4, 5, 6 | 2 | l.79-85 |
| Beat indicator | LED (`ui/led.js`) | on for 100 ms per click | off | l.89 |
| Slot min | Number field | any integer | 1 | l.93 |
| Slot max | Number field | any integer | 100 | l.94 |
| Slot display | LCD (`ui/lcd.js`) | integer or "---" | "---" | l.95 |
| Spin | Pad (`ui/pad.js`) | trigger | not applicable | l.96 |
| Instrument on/off | Switch (app level, stored as `on.metronome` in the project) | on, off | off | target description |
| Volume, pan, mute | Console strip, not in this tab | strip ranges (`docs/modules/console.md`) | strip defaults (`docs/modules/console.md`) | target description |
| Tempo, start, stop, swing | Global header, not in this tab | tempo 40 to 300 BPM, swing 0 to 70 % | tempo 120, swing 0 | target description |

The filter slider is linear in the source (step 1). A log scale is not in any source, so the knob is linear, 100 to 10000.

## State

One JSON-serialisable object. `slot.value` is the last displayed number, or `null` for "---" (the default). The example below shows a state after one spin.

```json
{
  "waveform": "sine",
  "filterHz": 1000,
  "noteLengthMs": 10,
  "octave": 2,
  "slot": { "min": 1, "max": 100, "value": 42 }
}
```

The on/off switch is stored in the project as `on.metronome`, not in this object. `schemaVersion` belongs to the project, and the shown tab is interface state; neither is in this object. `setState` accepts a partial object and fills missing fields with the defaults above (`slot.value` defaults to `null`).

## Audio graph

Per click, a fresh chain is built and discarded, as in the original:

```
OscillatorNode (type = waveform, frequency = 220 * 2^(octave-4))
  -> BiquadFilterNode (lowpass, frequency = filterHz)
  -> GainNode (gain = 0.5)
  -> output  (strip input from audio/mixer.js, handed over by the engine: gain, pan, EQ, reverb send, mute, solo -> master bus)
```

There is no envelope, no compressor and no analyser inside the voice. The slot machine has no audio. The module never touches `ctx.destination`.

## Transport behaviour

- Start: `onStart(time)` clears any pending click state. The first `onStep(0, time, stepDuration)` plays a click at `time`. When the switch is turned on while the transport runs, `onStart(time)` is called and the next `onStep` carries the current `stepIndex`.
- Each step: `onStep(stepIndex, time, stepDuration)` plays a click when `stepIndex % 4 === 0` (every quarter note, as in the source). Other steps are ignored. The oscillator starts at `time` and stops at `time + noteLengthMs / 1000`. `onStep` does audio only: it writes `{ stepIndex, time }` into a ring buffer, and a mounted interface reads it in the shared animation loop (`audio/meters.js`), lights the LED at `time` and clears it 100 ms later. Swing moves only off-beat 16ths, so quarter-note clicks are not shifted.
- Late steps: the transport skips a step that is more than 5 ms late. This module has no guard of its own.
- Stop: `onStop()` cancels clicks that are scheduled and ends every sounding click with a short fade (rule 8). The LED is cleared by the interface when the buffer is empty. A click already playing does not ring on after Stop.
- Tempo change: the header changes BPM. The next `onStep` uses the new `stepDuration`. Nothing in the module stores tempo, so a live change works at once (in the source it did nothing until restart).
- Tab hidden: audio does not depend on the DOM, so clicks keep playing. `unmount()` removes the animation-loop registration and any running spin timer. `mount()` redraws all controls from state, including the last slot value. With no mounted interface nobody reads the ring buffer, and an offline instance does no interface work.

## Defects fixed

| Defect | Report reference | Fix |
|---|---|---|
| `setInterval` timing with `osc.start()` without time, main-thread jitter, throttled in background tabs | weakness 1 (timing); l.132-133, l.122 | Click uses the transport lookahead scheduler and `time` on `audioContext.currentTime`. |
| Tempo read once at Start, live change ignored | l.130 | Tempo comes from the transport. |
| Empty or 0 BPM gives an infinite loop of oscillators | l.130, l.68 | Module has no tempo input; the transport keeps tempo within 40 to 300 BPM. |
| Stop does not cancel pending `osc.stop` or silence a playing blip | l.140 | `onStop()` cancels scheduled clicks and ends sounding ones with a short fade. |
| `AudioContext` created at load and `resume()` never called | l.99 | `audio/engine.js` creates the one context at page load and resumes it on the first user gesture anywhere on the page. |
| Spin can be pressed while running and creates overlapping intervals | l.144-155 | Spin is ignored while a spin runs, and the pad shows disabled. |
| Empty field shows NaN in the display | l.145-146, l.150 | Empty or non-numeric fields fall back to the defaults 1 and 100 at spin time. |
| Min greater than Max gives values outside the range | l.145-146 | Min and Max are swapped at spin time. |

Only fixes of defects that broke behaviour go to `CHANGELOG.md`. The clean-ups below do not.

## Rewrite and accessibility changes

| Change | Source reference | In RABBIT Studio |
|---|---|---|
| Global mutable state and repeated `getElementById` reads | l.99-101, l.109-116 | State object read by the module; no globals. |
| `parseInt` without radix | l.145-146 | `parseInt(value, 10)`. |
| Spin timer not cleaned up | l.152-154 | `unmount()` clears it. |
| Dutch labels ("Nootlengte", "Octaaf") | l.77, l.79 | English labels "Note length" and "Octave". |
| Controls without keyboard or ARIA support | whole file | Shared `ui/knob.js`, `ui/led.js`, `ui/lcd.js` and `ui/pad.js` with keyboard and ARIA support. |

## Kept on purpose

- Octave default 2, so the first click sounds at 55 Hz.
- The label "Octave" follows the original numbers, not note names (octave 4 gives 220 Hz).
- Abrupt oscillator stop with no envelope, including the click that this produces. The sound character stays; see open questions.
- A fresh oscillator, filter and gain per click.
- Click on quarter notes only, with no accent and no bar length.
- Fixed voice gain 0.5, short note lengths of 1 to 40 ms.
- Slot machine rules: 100 ms tick, 21 writes, last value stays, no payout, no sound. The result is the last random tick, not a separate "win".
- Beat indicator pulse of 100 ms.

## Layout

```
+-- METRONOME ----------------------------------------------[ ON ]--+
|                                                                    |
|  +-- CLICK ----------------------------+  +-- BEAT -------------+  |
|  |  WAVE  [ sine | saw | square ]      |  |                     |  |
|  |  OCTAVE [ 2 | 3 | 4 | 5 | 6 ]       |  |        (  O  )      |  |
|  |                                     |  |       beat LED      |  |
|  |   (FILTER)         (NOTE LENGTH)    |  |   tempo from header |  |
|  |   1000 Hz              10 ms        |  +---------------------+  |
|  +-------------------------------------+                           |
|                                                                    |
|  +-- SLOT MACHINE ----------------------------------------------+  |
|  |  MIN [  1 ]   MAX [ 100 ]    +-----------+   [ SPIN ]         |  |
|  |                              |    42     |                    |  |
|  |                              +-----------+                    |  |
|  +--------------------------------------------------------------+  |
+--------------------------------------------------------------------+
  page, accent red (beat LED, spin pad) and amber (LCD) come from styles/tokens.css,
  Michroma for titles, Barlow for labels
```

## Tests

All tests are pytest files in `02_backend/tests/`, run with `uv run pytest`. Module-level JavaScript checks run inside the page with Playwright for Python, with a stub clock where timers are involved. Live checks use `?test=1` with `window.__rabbit` and its `onStep` log.

- `createInstrument` returns all ten members of the contract (`id`, `title`, `ready`, `mount`, `unmount`, `onStart`, `onStop`, `onStep`, `getState`, `setState`) and an `id` of `metronome`.
- `getState` returns the default object; `setState` followed by `getState` round-trips the example state.
- `setState` with a partial object keeps the defaults for missing fields.
- `onStep` with `stepIndex` 0, 4, 8, 12 starts one oscillator each at the given `time`; indices 1, 2, 3, 5 start none.
- The started oscillator has frequency `220 * 2^(octave-4)` for octaves 2 to 6 (55, 110, 220, 440, 880 Hz).
- The oscillator stops at `time + noteLengthMs / 1000`.
- The chain is oscillator, lowpass biquad at `filterHz`, gain 0.5, then `output`; nothing connects to `ctx.destination`.
- `onStop` cancels scheduled clicks and ends sounding voices with a short fade; afterwards no voice sounds.
- Switching off calls `onStop`; switching on while the transport runs calls `onStart(time)` and the next click uses the current `stepIndex`.
- `onStep` touches no DOM and writes one `{ stepIndex, time }` entry to the ring buffer per click.
- A changed `stepDuration` between two steps changes nothing in the module and raises no error.
- Calls to `onStep` with the tab unmounted still produce sound and do not throw.
- `mount` then `unmount` leaves no running timer, no listener and no animation-loop registration.
- Live check: in `?test=1` mode the Metronome and another instrument log the same `stepIndex` at the same audio time, and a tempo change reaches the Metronome.
- Spin writes exactly 21 values in 2.1 s (stub clock), all within min and max, and leaves the last one.
- A second Spin during a running spin is ignored.
- Spin with min greater than max swaps them; an empty field uses the default.
- State returned after a spin contains the last value in `slot.value`.

## Open questions

- Pops from the abrupt end of each click: the report calls them a defect, but removing them changes the sound. Keep as is (default in this specification) or add a short release ramp? The short fade of `onStop` (rule 8) does not decide this.
- The original has no accent; a 16-step metronome could accent beat 1 of the bar. No source asks for it, so it is not included.
- The slot machine is outside the transport; should its state be saved with the project? The specification saves it (`slot`), which is not a source feature.
- Should the filter knob be logarithmic for easier use? The source slider is linear, so the specification keeps it linear.
