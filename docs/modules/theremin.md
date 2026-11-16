# Theremin

## Sources

One source file feeds this module. The report is `/tmp/claude-1000/-home-hanmul-Workshops-workshop-rapid-prototyping/b3a1cb86-51fa-4dc8-b6d3-abb1f8743206/scratchpad/inventory/report_CP5_FD_20251104_theremin01.json`. The original is `/tmp/claude-1000/-home-hanmul-Workshops-workshop-rapid-prototyping/b3a1cb86-51fa-4dc8-b6d3-abb1f8743206/scratchpad/intropakket/IntroPakket/CP5_FD_20251104_theremin01.html` (373 lines, Dutch, an "older CP5" file). In the table below it is called TH. The report was read in full. The HTML markup (l.55-130) was opened to check the control ranges, defaults and initial texts; all matched the report. The HTML was not run.

## Purpose

The Theremin is a monophonic instrument that you play with a bright object in front of the webcam. The vertical position of the brightest area sets the pitch, and the horizontal position sets the volume. It is not a sequencer: it ignores `onStep` and has no tempo or pattern. Its audio goes to its own channel strip, like every other instrument. `audio/mixer.js` builds the strip, the engine hands its input to the instrument as `output`, and the Console shows its controls. The camera works only on `http://localhost`, `http://127.0.0.1` or an HTTPS address (see docs/plan.md, section Docker).

## Feature parity

| Feature in the source | Source | In RABBIT Studio |
|---|---|---|
| Start button: asks for the webcam (facingMode "user", ideal 640x480, no audio), plays the hidden video, creates the AudioContext, resumes it, starts the rAF loop; on error an `alert` and Start is enabled again (l.91, l.189, l.324-338) | TH | Kept as the "Camera on" button of the instrument. The AudioContext is not created or resumed here: `audio/engine.js` creates the one context at page load and resumes it on the first user gesture anywhere on the page, so the Theremin sounds even if the transport never started. The camera request still needs a click, so the button stays. The error `alert` is replaced by a message on the LCD of the plate (permission denied, no camera, no secure context are told apart). The wall-clock start and the countdown of other files do not exist in this source |
| Stop button: cancels rAF, `ac.suspend()`, stops all camera tracks, nulls the stream, toggles the button states; also bound to `beforeunload` (l.92, l.340-352, l.370) | TH | Kept as the "Camera off" button. It cancels the loop, stops the camera tracks, nulls the stream and ramps the gain to 0. It never calls `suspend()`: that would silence every instrument. The `beforeunload` cleanup moves to `pagehide` and to the end of the instrument (see Defects fixed) |
| Recalibrate button: only shows an `alert` that points to the sigma slider, no calibration (l.93, l.354-360) | TH | Kept as a small button with the same label meaning, "Recalibrate threshold". It still does no calibration; it shows the same hint on the LCD instead of an `alert`. Whether to keep a button that does nothing is an open question |
| Live preview canvas `#view`, 640x480 default, resized to the video size; raw unmirrored frame each rAF; 16 px circle at the centroid, composite "screen", line width 2; ring rgba(107,196,255,.9) when found, rgba(255,107,107,.7) when not (l.197, l.256-264) | TH | Kept. Frame and ring are unmirrored as in the source. The ring colours change to the RABBIT look: found = the amber token, not found = the accent red token (both come from `styles/tokens.css`; no colour value in any notation appears in the module, rule 6). Drawing happens only while the tab is mounted; analysis does not depend on it |
| HUD: horizontal line `#barY` and vertical line `#barX` (2 px each) placed in % of the centroid, start at 50% and 50%; readout "freq: X.X Hz \| vol: X.XX \| luminantie σ: X.X" showing the targets; initial text "freq: - Hz \| vol: - \| luminantie: -" (l.80-82, l.252-253, l.318) | TH | Kept, moved onto the `ui/lcd.js` readout and two hairline lines over the canvas. Text is English: "freq 440.0 Hz \| vol 0.25 \| sigma 0.5". It still shows targets, not measured audio values. The mismatch between the initial and the running text is removed by using one format (see Rewrite and accessibility changes) |
| Detection `analyzeFrame`: offscreen canvas max(128, round(0.25*vw)) x max(96, round(0.25*vh)), so 160x120 for 640x480; Rec.709 luma 0.2126R+0.7152G+0.0722B; whole-frame mean and standard deviation; threshold = mean + threshK*std; centroid weighted by (Y - thr) for pixels with Y > thr; found = weight sum > 1e-3; if nothing is found the centroid is the frame centre (l.208-267) | TH | Kept unchanged. This is the sound and play character. The aspect-ratio distortion on non-4:3 cameras is listed under Open questions |
| Pitch mapping `expoMap`: freq = minF * (maxF/minF)^(1 - yn); top of frame = maxF, bottom = minF; continuous, no quantisation (l.270-274, l.293) | TH | Kept unchanged. Min and max are clamped so that min is less than max (see Defects fixed) |
| Volume mapping: target gain = found ? (0.05 + 0.95*xn) * maxGain : 0; left edge 5 % of the maximum, right edge 100 % (l.297) | TH | Kept unchanged. The result goes to the instrument gain node, then to the channel strip. The strip fader (default gain owned by docs/modules/shell.md and docs/modules/console.md) is a second, separate volume: "Max. volume" scales the playing range, the strip fader scales the whole instrument |
| Smoothing: `setTargetAtTime` on frequency and gain with tau = max(0.001, smoothMs/1000) s (so 0 ms still gives 0.001 s); try/catch with a `.value` fallback "for Safari" (l.289-290, l.304, l.311) | TH | Kept. The `try/catch` fallback is dropped because `setTargetAtTime` is supported everywhere in current browsers (the report calls the fallback unreachable) |
| Collapsible help "Hoe werkt de detectie?" with the algorithm; a tip that the page needs https or localhost for the camera (l.119, l.123-126) | TH | Kept as a collapsible help block, translated to English, in the style of the plate. The secure-context tip stays; `http://localhost` counts as secure. The Python `http.server` example of the tip is dropped as it is not about the instrument |
| Value labels next to the sliders: "150-1200 Hz" for the frequency range, and the current value for smoothing, sigma and max. volume (l.97, l.105, l.110, l.115) | TH | Kept. The shared `ui/knob.js` shows the value of each knob. The two frequency knobs each show their own value |
| Live variables: `threshK` is updated by a listener and re-read in `start()`; max. volume, min and max frequency and smoothing are read from the sliders every frame (l.287-296, l.329, l.367) | TH | The values live in one state object, not in DOM values. Every frame reads that object, so a knob change and `setState` act on the next frame |
| Responsive layout: below 860 px the two panels stack into one column (l.62-64) | TH | Kept, with the breakpoint of the app shell. Inside the app the Theremin is a tab, so the two racks stack when the tab is narrower than 860 px. The smallest supported width is 640 px; there are no rules or tests for narrower screens |
| Not present in the source: step sequencer, other voices, tempo, multiple channels, scales or note names, recording, sample loading, export, presets, keyboard or MIDI input, save/load | TH | Not added. The only additions are those the target application names: the on/off switch (stored in the project as `on.theremin`), the Console strip (volume, pan, EQ, reverb send, mute, solo), the global transport in the header (start, stop, tempo, swing; the Theremin ignores all of them) and project save and load of the state |

## Controls

| Control | Component type | Range | Default | Source |
|---|---|---|---|---|
| On/off switch of the instrument (stored in the project as `on.theremin`, not in the instrument state) | `ui/led.js` with a toggle button | on, off | off | Target application |
| Camera on | button | click | enabled while off | TH l.91 |
| Camera off | button | click | disabled until the camera runs | TH l.92 |
| Recalibrate threshold | button | click | enabled | TH l.93 |
| Min frequency | `ui/knob.js` | 50 to 1000 Hz, step 1 | 150 | TH l.99 |
| Max frequency | `ui/knob.js` | 300 to 3000 Hz, step 1 | 1200 | TH l.100 |
| Smoothing | `ui/knob.js` | 0 to 250 ms, step 5 | 50 | TH l.106 |
| Threshold sensitivity (sigma; lower is more sensitive) | `ui/knob.js` | 0 to 2, step 0.05 | 0.5 | TH l.111 |
| Max. volume | `ui/knob.js` | 0 to 1, step 0.01 | 0.5 | TH l.116 |
| Output level | `ui/vu.js`, 10 segments, read from the instrument gain node; the dB range passed to it is the meter range owned by docs/modules/shell.md and docs/modules/console.md | 0 to 10 segments | 0 | RABBIT look; the source has no meter |
| Detection status | `ui/led.js` (lit when a target is found) | on, off | off | TH l.259 (found and not found are two ring colours) |
| Readout | `ui/lcd.js` | text | "freq - Hz \| vol - \| sigma -" | TH l.82, l.318 |
| Volume, pan, EQ, reverb send, mute, solo | Console strip, not on this plate | by the Console | by the Console | Target application |

The source has no fader, pad or beat-related control. `ui/fader.js` and `ui/pad.js` are not used by this module. The source uses no pointer or keyboard input beyond the sliders and buttons; the main interaction is the bright object in front of the camera.

## State

One plain JSON object. The camera state is not saved, because a camera can start only after a click. The on/off switch is not part of this object: the project stores it as `on.theremin`. Which tab is shown is interface state and never enters the project. `schemaVersion` belongs to the project, not to this object.

```json
{
  "minFreq": 150,
  "maxFreq": 1200,
  "smoothMs": 50,
  "threshK": 0.5,
  "gainMax": 0.5
}
```

This is the default state. `setState` accepts a partial object, clamps each number to the range in the table above and ignores unknown keys. A project that holds no Theremin state gives the defaults.

## Audio graph

There is one voice. Per voice: `OscillatorNode (sine)` to `GainNode (instrument gain, starts at 0)` to `output`. The `output` is the strip input that `audio/mixer.js` built (gain, pan, EQ, reverb send, mute, solo), which feeds the master bus. The Console builds no strip and the instrument never connects to `ctx.destination`. The module adds no filter, compressor or effect of its own. The VU meter reads the instrument gain node through the shared `ui/vu.js`; this is a read-only tap and does not change the signal. The meter registers with the one animation loop in `audio/meters.js` when the plate is mounted and has no loop of its own. The oscillator starts once in `createInstrument` and runs until the page closes; silence comes from gain 0 (as in the source). The context is the one `ctx` that `createInstrument` receives. The module does not use `fx/reverb.js`, `fx/delay.js` or `audio/noise.js`; the reverb send lives on the Console strip.

## Transport behaviour

`onStart` and `onStop`: nothing happens. This is the exception to rule 8 in docs/plan.md, section Rules that keep the structure: the Theremin is played live, so the transport does not silence it. Only its own switch (or Camera off) does. Its sound depends only on that switch and on the camera buttons.

Each step: `onStep(stepIndex, time, stepDuration)` is called and returns at once. The Theremin has no steps, writes nothing to the ring buffer and has no late-step guard (the guard exists once, in the transport; see docs/plan.md, section Transport).

Tempo change and swing change: no effect. The instrument does not subscribe to the transport.

Mixdown: the Theremin is left out, because it depends on a live camera. The status line of the mixdown says so.

Switch on: the switch is stored in the project as `on.theremin`; the instrument is armed. Switching on while the transport runs calls `onStart(time)`, which does nothing here. Sound starts when the camera runs. Camera on: ask for the webcam, play the off-DOM video, start the rAF analysis loop. Switch off or Camera off: cancel the loop, stop all camera tracks, ramp the gain to 0.

Tab hidden inside the app: `unmount()` removes only the DOM (plate, canvas, HUD). The video element, the offscreen analysis canvas, the oscillator and the gain node are created in `createInstrument`, not in `mount`, and the rAF loop is owned by the instrument, not by the plate, so a switched-on Theremin keeps playing. The preview canvas and the HUD are drawn only while mounted.

Browser tab hidden: the browser throttles or pauses rAF, so no new targets are computed and the gain and frequency keep their last target (source behaviour). See Open questions.

## Defects fixed

Only fixes of defects that broke behaviour are listed here, and each one gets a line in `CHANGELOG.md`. Clean-ups of the rewrite are listed under Rewrite and accessibility changes.

| Defect in the source | Report line | Fix |
|---|---|---|
| `stop()` calls `ac.suspend()` on the context; in the app this would silence all instruments | l.343-345 | Stop never suspends or closes the shared context. It stops the camera tracks and ramps the gain to 0 |
| Start after Stop works only because the context is resumed | l.330 | `audio/engine.js` resumes the shared context on the first user gesture anywhere on the page, not this module; Camera on does not touch it |
| AudioContext created inside the click handler after an `await`, which can break the gesture rule in Safari | l.327-329 | `audio/engine.js` creates the one context at page load; the instrument never creates one |
| Min frequency can exceed max frequency, which inverts the pitch; min = max flattens it | l.99-100, l.272-273 | `setState` and both knobs keep min at least 1 Hz below max, by moving the other value |
| No check that `navigator.mediaDevices` exists on an insecure origin; permission denied and no camera are not told apart; `alert` blocks | l.190, l.334 | Check `navigator.mediaDevices` first, tell the three cases apart, show the message on the LCD |
| `beforeunload` is the wrong event for cleanup on mobile | l.370 | Camera tracks are also stopped on `pagehide` and when the instrument is switched off |
## Rewrite and accessibility changes

These changes do not fix a defect that broke behaviour. They are not in `CHANGELOG.md`.

| Item in the source | Report line | Change |
|---|---|---|
| Misleading `try/catch` around `setTargetAtTime` | l.303-308 | Removed (see feature parity) |
| Mismatch between initial and running readout text | l.82, l.318 | One format for both |
| Labels without `for=`, so sliders are not associated | l.97, l.105, l.110, l.115 | Each knob gets an accessible name through the shared `ui/knob.js` |
| New `Float32Array` and a `getImageData` read every frame cause garbage-collection churn | l.211, l.216 | The luma buffer is allocated once and reused; the result of the detection does not change |
| Settings are read from DOM slider values each frame, so UI and audio state are mixed | l.287-296 | One state object holds the settings; the knobs write to it and the frame loop reads it |
| The HUD is `aria-hidden`, so the readout values are not announced | l.79 | The LCD readout is not hidden from assistive technology; the canvas keeps its label |
| Interface strings are Dutch only | l.70-126 | All strings are English, as the target application requires |

## Kept on purpose

- Sine wave, no filter, no effect, no limiter. The tone is plain.
- Exponential pitch map with no note quantisation; top of the frame is the highest note.
- Volume floor of 5 % of the maximum at the left edge, and gain 0 only when nothing is found.
- "Found" is true on almost any non-flat frame, so the tone seldom goes silent when the hand leaves. This is a sound and play quirk of the original, as are the missing hysteresis and noise gate, and the fact that a bright background steals the centroid.
- Unmirrored video and unmirrored x-to-volume mapping.
- Detection on a 160x120 frame with mean + k*sigma and (Y - thr) weighting, including the unchanged analysis canvas size rule for other aspect ratios.
- `setTargetAtTime` smoothing with tau = max(0.001, smoothMs/1000).
- The oscillator runs for the whole session and silence comes from gain 0.
- The Recalibrate button gives a hint only.
- The target values, not measured values, shown in the readout.
- The analysis stays tied to rAF; `requestVideoFrameCallback` is not used.
- The default max. volume of 0.5, because a full-scale sine is loud and the source has no limiter.

## Layout

```
+--------------------------------------------------------------------------+
| RABBIT STUDIO   [> start] [] stop   BPM 120   SWING 0    (global header) |
+--------------------------------------------------------------------------+
| tabs: beat | synth | slicer | [THEREMIN] | metronome | console            |
+--------------------------------------------------------------------------+
| THEREMIN                                               (o) ON   [ON/OFF] |
| +------------------------------+  +-----------------------------------+  |
| | CAMERA                       |  | CONTROL                           |  |
| | +--------------------------+ |  |  [Camera on] [Camera off]         |  |
| | |        |                 | |  |  [Recalibrate threshold]          |  |
| | |   ---- O ---- (ring)     | |  |                                   |  |
| | |        |   crosshair     | |  |  (MIN HZ) (MAX HZ) (SMOOTH)       |  |
| | +--------------------------+ |  |  (SIGMA)  (MAX VOL)    knobs      |  |
| | LCD: freq 440.0 Hz | vol 0.25|  |                                   |  |
| | (o) target found             |  |  VU |||||||||| (10 segments)       |  |
| +------------------------------+  +-----------------------------------+  |
| [v] How does the detection work?   (help, collapsible)                   |
+--------------------------------------------------------------------------+
```

Colours (page, accent red, amber) and fonts (Michroma for titles, Barlow for text) come from `styles/tokens.css`. Below 860 px the two racks stack. The smallest supported width is 640 px.

## Tests

All tests are pytest files in `02_backend/tests/` (for this module `test_theremin.py`), run with `uv run pytest`. Module-level JavaScript checks run inside the page with Playwright for Python; live checks use the `?test=1` mode with `window.__rabbit` and its `onStep` log.

- `defaults`: `createInstrument` returns every member of the contract in docs/plan.md, section The instrument contract, and `getState()` equals the default state (without `on`).
- `state-roundtrip`: `setState(getState())` changes nothing, and the result is JSON-serialisable.
- `state-clamp`: out-of-range and unknown keys in `setState` are clamped or ignored.
- `freq-order`: min above max is corrected so that min stays below max.
- `expo-map`: the top row gives maxF, the bottom row gives minF, and the middle gives the geometric mean.
- `volume-map`: found at the left edge gives 0.05*maxGain, at the right edge gives maxGain, and not found gives 0.
- `detect-centroid`: a synthetic frame with one bright square gives its centroid; a flat frame gives the frame centre and a threshold above the mean.
- `no-step`: `onStep` and `onStart` change no audio parameter.
- `stop-exception`: header Stop and `onStop` leave a sounding Theremin sounding (rule 8 exception); its own switch, set through `on.theremin`, ramps the gain to 0.
- `switch-in-project`: the switch is read from `on.theremin` and `getState()` holds no `on` key.
- `mixdown-excluded`: the offline mixdown does not create the Theremin and the status line says so.
- `output-routing`: the oscillator reaches `output` only through the gain node, the only other connection is the read-only meter tap, and nothing connects to `ctx.destination`.
- `no-suspend`: Camera off and switch off never call `suspend()` or `close()` on the context.
- `hidden-tab`: after `unmount()` the loop and the gain keep running; after a second `mount()` the preview draws again.
- `camera-release`: Camera off stops every track, and `pagehide` does the same.
- `camera-errors`: no `mediaDevices`, permission denied and no camera show three different LCD messages and leave the Camera on button enabled.
- `e2e-switch`: with Chromium's fake camera, switching on and pressing Camera on gives permission granted and a playing video, and Camera off enables Camera on again. The frame analysis is tested on a synthetic image (`detect-centroid`), not through the camera.

## Open questions

1. The video and the x-to-volume mapping are unmirrored, so the hand moves right while the image moves left (report l.257, l.297). This spec keeps it because it is a quirk of the original. Mirroring is a one-line change if you prefer.
2. A hidden browser tab pauses rAF, so the last tone holds. Should the gain ramp to 0 on `visibilitychange`? The source does not.
3. The analysis canvas distorts pixels for cameras that are not 4:3 (report l.201-202). Fix it, or keep it? The overlay is not affected.
4. Should the Recalibrate button stay, as a button that only shows a hint? The source label promises more than it does.
5. The module has no teardown hook, so the oscillator and the loop end only when the page closes. Add a destroy call to the instrument API if projects are switched often.
6. Camera frame rate and the exact resolution the browser grants are not fixed by the source (ideal 640x480 only). Value for the test fake camera: TBD.
7. Does the project save the camera choice (device id)? The source has no device choice, so this spec saves none.
8. The instrument needs a camera and a visible bright object; the source has no keyboard or pointer alternative (report accessibility note). Add one, or accept this?
