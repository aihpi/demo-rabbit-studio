# Slicer

## Sources

- Report: `/tmp/claude-1000/-home-hanmul-Workshops-workshop-rapid-prototyping/b3a1cb86-51fa-4dc8-b6d3-abb1f8743206/scratchpad/inventory/report_CP5_TT_20251118_Slicer5_update20260524.json`
- Original (read as data, not run): `/tmp/claude-1000/-home-hanmul-Workshops-workshop-rapid-prototyping/b3a1cb86-51fa-4dc8-b6d3-abb1f8743206/scratchpad/intropakket/IntroPakket/CP5_TT_20251118_Slicer5_update20260524.html` ("CP5" file, 513 lines, "Random Slice Sampler"). This is the only source of the module. Line numbers in this document refer to that file, as cited in the report.
- Module folder: `01_frontend/instruments/slicer/`, id `slicer`.

## Purpose

The Slicer loads one audio file, cuts it into equal-length slices and plays randomly chosen slices on the 16th-note grid of the global transport. Every step rolls a die (density) and picks a random slice, so the pattern is never the same twice. A shared filter, a per-voice ADSR envelope and a convolution reverb shape the sound. The audio goes to the Slicer's channel strip on the Console.

## Feature parity

Every entry of the report's "features" array is listed, in report order. "Console" means the Slicer's channel strip on the Console. "Header" means the global transport in the page header.

| Feature in the source | Source | In RABBIT Studio |
|---|---|---|
| Audio file loading: file input `audio/*`, decoded with `decodeAudioData`, one sample at a time, status text "Laden..." then "Sample geladen (X.XX s)" | SL 92, 295-305 | Kept. The file is uploaded to the backend, the state stores its id (`sampleId`), and the instrument fetches and decodes it. One sample at a time. Status text in English on the LCD ("Loading...", "Sample loaded (X.XX s)"). A decode error now shows "Cannot decode file" and keeps the previous sample (see Defects fixed). |
| Reset button: stops the sequencer, clears buffer and slices, redraws placeholder, clears file input; scheduled voices are not cancelled | SL 93, 386-392 | Kept as "Clear sample". It clears `sampleId`, buffer and slices, redraws the placeholder and clears the file input. It no longer stops the global transport (the header owns that); it fades out active voices. |
| Start (ge-syncd) button: needs a sample, resumes the context, waits for wall-clock seconds%4==0, then starts | SL 97, 406-422 | Moves to the header (start). The wall-clock wait and its countdown are removed by decision. The "load a sample first" rule stays: with no sample, `onStep` does nothing and the LCD shows "Load a sample". |
| Stop button: clears the 25 ms interval; scheduled notes still sound; a pending start timeout is not cancelled | SL 98, 424, 434-438 | Moves to the header (stop). The instrument's `onStop()` now also silences voices that are still playing (see Defects fixed). The pending start timeout no longer exists (removed by decision). |
| Sync status line: "Wacht op sample", "Laden", countdown "Start over X.XX s", "Sequencer loopt (ge-syncd)", "Gestopt", "Reset" | SL 99 | Kept as an LCD status line with English texts: "No sample", "Loading...", "Running", "Stopped", "Cleared". The countdown text is removed by decision. "ge-syncd" is dropped because the header transport is the sync. |
| Random slicing: slice count = max(1, floor(duration / sliceMs)), equal length, remainder shorter than one slice never played; slice length live-adjustable and recomputes slices and redraws | SL 307-317 | Kept as it is, including the unplayed remainder. Slices stay random (uniform `Math.random`, no seed). |
| Step grid: fixed 16th-note tick, 15/BPM s (125 ms at 120 BPM); each tick fires with probability density and picks a uniformly random slice; no pattern, accents or swing | SL 209, 443-444 | The grid now comes from the global transport: `onStep(stepIndex, time, stepDuration)` is called for each 16th step. Tempo (BPM) and swing move to the header; the instrument has no own BPM slider. Slice choice and density roll stay random per step. No pattern, no accents. Swing is applied by the transport to `time`, so the Slicer needs no swing code; this is an effect of the header feature, not a new Slicer feature. |
| Per-note jitter: start offset = slice start + uniform(-jitter, +jitter) ms, clamped to [0, duration - 0.005] | SL 453-454 | Kept. |
| Note length = 0.95 x slice duration; source started with duration dur+R+0.02 s, stopped at noteEnd+R+0.03 | SL 455, 481-483 | Kept. The redundant `Math.min` is dropped. The note length is a fraction of the slice duration, not of the step. |
| Per-voice ADSR on a GainNode: 0.0001 to 1.0 over max(1 ms, A), to S over D, hold S until noteEnd, release to 0.0001 over R; peak always 1.0 | SL 461-475 | Kept, including the fixed peak of 1.0 (no velocity). The A+D overrun case is guarded (see Defects fixed). |
| Shared biquad filter: lowpass, highpass, bandpass, notch; cutoff; Q | SL 140-145, 398 | Kept. One shared filter for all voices. |
| Parallel reverb: filter to dry gain (fixed 1.0) and filter to convolver to wet gain; generated stereo noise impulse response with (1 - i/length)^2 decay; length user-set; rebuilt on every slider input | SL 279-292, 402 | Kept, but built with the shared `fx/reverb.js`: `createReverb(ctx, { decay, preDelay, power, toneHz, mix })` called with `decay` = size, `preDelay` 0 and `power` 2, which gives the source's (1 - i/length)^2 noise impulse response. Rebuilt after 150 ms without further input instead of on every input event. Wet stays additive (dry fixed at 1.0). This is the Slicer's own reverb; the Console's reverb send is a separate, additional send. |
| Master volume gain | SL 403 | Moves to the Console strip (gain). The Slicer has no volume control of its own. |
| VU meter: RMS of the 1024-sample buffer, -60..0 dB, numeric dB readout with 1 decimal ("-inf" below -80 dB), red "CLIP" when RMS dB > -1; runs forever on requestAnimationFrame | SL 486-503 | Kept as the shared `ui/vu.js` (10 segments; the Slicer passes the range -60..0 dB), with the dB readout on `ui/lcd.js` (same format, "-inf" below -80 dB) and CLIP on a `ui/led.js` LED. The RMS test stays (the sound character is the same); see open questions. The meter registers with the shared animation loop in `audio/meters.js` on mount and deregisters on unmount; the Slicer has no loop of its own. A second, final-level meter is on the Console strip. |
| Waveform and slice canvas: left channel only, min/max per pixel column, alternating slice bands, slice separator lines, 3 px bottom bar, placeholder text when empty; no playhead; not drawn at load; no resize handler | SL 133, 319-384 | Kept as a canvas plate. Colours are read from CSS variables (RABBIT amber for the waveform, separators and bottom bar), not hard-coded. It is drawn on mount and whenever sample or slice length changes, and on resize. Left channel only stays. The placeholder is drawn at load, so the plate is never a blank box. No playhead and no indication of the playing slice. |
| Value readouts for every slider via `mirror()`: BPM integer, %, ms, Hz integer, Q one decimal, reverb size one decimal + " s" | SL 228-250 | Kept. Each knob shows its value on the LCD with the same formats. The BPM readout moves to the header. |
| No presets, recording, export, keyboard shortcuts, MIDI, multiple samples, per-slice control, settings save | SL (report, last feature) | Stays absent. The only persistence is the project save and load by name that applies to all instruments (target application). |

Where the sources disagree: there is only one source file, so no implementation choice between sources is needed.

## Controls

All numeric controls are `ui/knob.js` knobs with a `ui/lcd.js` readout. Ranges and defaults come from the report (line numbers are of the original). Step is 1 where the report gives none.

| Control | Component type | Range | Default | Source |
|---|---|---|---|---|
| Sample file | file button (native input `audio/*`) | one audio file | none | SL 92 |
| Clear sample | button (`ui/pad.js`) | n/a | n/a | SL 93 |
| On/off switch | switch plus LED (`ui/led.js`) | on, off | off | stored in the project as `on.slicer`, not in the instrument state |
| Density | knob | 0-100 % | 75 | SL 121 |
| Slice length | knob | 40-1000 ms, step 10 | 180 | SL 126 |
| Jitter | knob | 0-200 ms, step 5 | 20 | SL 131 |
| Filter type | select (four segment pads) | lowpass, highpass, bandpass, notch | lowpass | SL 140-145 |
| Cutoff | knob | 50-20000 Hz, step 1 | 1200 | SL 151 |
| Resonance (Q) | knob | 0-20, step 0.1 | 0.8 | SL 156 |
| Attack | knob | 0-500 ms, step 1 | 8 | SL 165 |
| Decay | knob | 0-2000 ms, step 5 | 60 | SL 170 |
| Sustain | knob | 0-100 % of 1.0 | 60 | SL 175 |
| Release | knob | 0-3000 ms, step 10 | 120 | SL 180 |
| Reverb wet | knob | 0-100 % | 18 | SL 185 |
| Reverb size | knob | 0.3-6 s, step 0.1 | 2.2 | SL 190 |
| VU meter | `ui/vu.js`, 10 segments, read only | -60..0 dB | n/a | SL 486-503 |
| CLIP indicator | `ui/led.js`, read only | lit when RMS > -1 dB | off | SL 495-499 |

Removed from the instrument: Volume (SL 107, 0-100 %, default 90) goes to the Console strip gain; Tempo BPM (SL 116, 40-220, default 120) goes to the header, where the transport range is 40 to 300 BPM (see `docs/plan.md`, section Transport); Start, Stop and the Start button's wall-clock sync go to the header or are removed by decision.

## State

One plain JSON object returned by `getState`. The on/off switch is not part of it: the project stores it as `on.slicer`. `sampleId` is the backend id of the uploaded audio file, or `null`. Units match the controls table. Slices and the buffer are derived, not stored. Random choices are not stored.

```json
{
  "sampleId": "a41f9c02",
  "sampleName": "breakbeat.wav",
  "density": 75,
  "sliceMs": 180,
  "jitterMs": 20,
  "filter": { "type": "lowpass", "cutoff": 1200, "q": 0.8 },
  "env": { "attackMs": 8, "decayMs": 60, "sustain": 60, "releaseMs": 120 },
  "reverb": { "wet": 18, "sizeS": 2.2 }
}
```

`setState` applies all values to the knobs and audio params, and if `sampleId` changed it fetches and decodes the file. A state with `sampleId: null` clears the sample. Missing keys fall back to the defaults above.

## Audio graph

Created once in `createInstrument`, independent of the DOM (the AudioContext is the shared one).

Per voice, created for each scheduled note (2 nodes, no polyphony limit):

`AudioBufferSourceNode` (shared decoded buffer, stereo kept) to `GainNode` vGain (ADSR) to shared `filter`.

Shared chain to the output:

- `filter` (BiquadFilter, defaults lowpass, 1200 Hz, Q 0.8, then the state values) to `dryGain` (fixed 1.0) to `mix`
- `filter` to `fx/reverb.js` convolver to `wetGain` (state wet) to `mix`
- `mix` to `output` (the input of the Slicer's channel strip, which `buildMixer` in `audio/mixer.js` builds and the engine hands over)
- `mix` to `AnalyserNode` (fftSize 1024) for the VU and the CLIP LED (read only, no output connection)

There is no master gain inside the module (the Console strip gain replaces SL 403) and no compressor or limiter, as in the original. The filter is shared and not per voice.

Filter, wet and reverb values are written to the nodes at creation from the state, so nothing is lost before a file is loaded.

## Transport behaviour

- Start: the header transport calls `onStart(time)` on a switched-on instrument. It does not wait for a wall-clock second and shows no countdown. If no sample is loaded, steps are skipped.
- Stop: `onStop()` cancels the Slicer's scheduled voices and ends sounding voices with a short fade, by ramping their gain to 0 over 30 ms and stopping their sources. Reverb tails may ring out. Voices that are scheduled but not yet started are stopped at their start time. The Slicer keeps a list of live voices for this.
- Each step: `onStep(stepIndex, time, stepDuration)`. If there is no sample, return. The Slicer has no late-step guard; the transport skips a step more than 5 ms late (see `docs/plan.md`, section Transport). Otherwise roll `Math.random() <= density/100`; on success pick a uniformly random slice, add uniform jitter, and schedule one voice at `time` with `src.start(time, offset, dur)`. The note length is 0.95 x slice duration, independent of `stepDuration`. `stepIndex` is not used for slice choice.
- Tempo change: the transport owns tempo and swing. The Slicer reads nothing about BPM, so a change applies from the next `onStep` call. Already scheduled voices keep their time, as in the original.
- Tab hidden: the Slicer keeps playing as long as it is switched on, because the scheduler lives in the transport and the audio nodes do not depend on mounted DOM. `unmount()` stops the waveform and VU loops and removes DOM listeners only. `mount(el)` redraws the waveform and registers the VU with the shared loop. The Slicer has no playhead, so it writes nothing to a ring buffer. The transport runs its timer in a Web Worker with a 0.2 s lookahead, so a hidden tab holds time.
- Switch off: the project value `on.slicer` becomes false, `onStop` runs and `onStep` is no longer called. Switching on while the transport runs calls `onStart(time)`, and the Slicer plays from the next `onStep`.
- Mixdown: the Slicer is part of the WAV mixdown, rendered through an offline transport. Its settings are the same as what was heard, but the random slices are not.

## Defects fixed

Only defects that break behaviour are fixed.

- Filter type, cutoff, Q, wet and reverb size handlers throw before a file is loaded, and ensureAudio then overwrites the choice with 1200 Hz, Q 0.8, lowpass, wet 0.18 (SL 398-402, 258, 261). Fix: the audio nodes exist from creation and are initialised from the state.
- Stop and Reset do not silence voices already scheduled (SL 434-438, 386-392). Fix: `onStop()` and Clear fade out live voices.
- Stop during the countdown does not cancel the start timeout (SL 421). Gone, because the countdown is removed by decision.
- Start-only sync on the system clock (`Date`), never rechecked, with drift between tabs, uncompensated output latency and per-tab BPM offsets (SL 411-421, 395, 441). Gone: the wall-clock start is removed by decision and the header transport is the single shared clock and tempo.
- Late scheduler or late timeout fires missed ticks in a burst at once (SL 442-448). Fix: the transport skips a step more than 5 ms late; the Slicer has no guard of its own.
- Envelope overrun: if A+D exceeds the note length, the ramp to S overlaps `setValueAtTime(S, noteEnd)` (SL 471, 474). Fix: the decay ramp is clipped to the note length so the automation events stay in order.
- `decodeAudioData` has no try/catch, so a bad file leaves the status on "Loading..." with an unhandled rejection (SL 299-301). Fix: catch, show "Cannot decode file", keep the previous sample.
- Loading a new file while running swaps buffer and slices under the scheduler (report weaknesses, SL 296-312). Fix: the new buffer and slices are built first and swapped in one step.
- A file chosen without a user gesture can leave the context suspended until Start (report weaknesses). Fix: `audio/engine.js` creates the one AudioContext at page load and resumes it on the first user gesture anywhere on the page; the Slicer does not touch it.
- Reverb size slider rebuilds a noise impulse synchronously on every input (SL 279-292, 402), which causes UI jank, and replacing the convolver buffer during playback can click. Fix: the shared `fx/reverb.js` rebuilds after 150 ms without further input and owns the buffer swap.
- Waveform not drawn at load and not redrawn on resize (SL 319-322). Fix: drawn on mount, on load, on slice change and on resize.
- Duplicate schedulers and a start-only "Start" button (SL 97, 426-432). Gone: there is no own scheduler; the transport has start and stop.
- VU loop never cancelled (SL 487-503). Fix: the VU registers with the shared loop in `audio/meters.js` on mount and deregisters on unmount.

Each fix above gets one line in `CHANGELOG.md`.

## Rewrite and accessibility changes

These changes come with the rewrite. They are not behaviour defects and are not listed in `CHANGELOG.md`.

- Dead variables `sliceMs` and `jitterMs`, empty jitter handler, unused CSS variables, hard-coded canvas colours, `vGain.gain.value=0` redundancy, two state patterns (globals and DOM reads), global mutable script state (SL 198-212, 207-208, 397, 14, 18-20, 323-383, 459, 468). Cleaned up because the module is rewritten as one state object.
- Layout quirks (no `box-sizing`, empty grid cells, spacer div at SL 146) and accessibility gaps (canvas without text alternative, readouts not `aria-live`, VU without `role=meter`, CLIP by colour only). The new layout removes the quirks, and the canvas gets an `aria-label`, the LCD an `aria-live` region and the VU `role="meter"`.

## Kept on purpose

- Random slices stay random: uniform `Math.random`, no seed, no stored pattern, no shared pattern between instances.
- Density is a per-step probability, and the unplayed remainder after the last full slice stays unplayed.
- Wet reverb is additive (dry fixed at 1.0), not a crossfade.
- One shared filter for all voices, no per-voice filter, no compressor or limiter; overlapping voices peak at 1.0 each and can sum above 1.0 (clipping at high density is part of the sound).
- Peak 1.0 for every voice (no velocity), note length 0.95 x slice duration, release tail added to the source stop time.
- The reverb impulse is random noise with (1 - i/length)^2 decay, so each rebuild sounds slightly different.
- The cutoff scale is linear over 50-20000 Hz.
- The waveform shows the left channel only.
- The VU and CLIP use RMS, not a peak detector.
- No polyphony limit.

## Layout

The module has three plates inside a RABBIT chassis (page, red accent and amber from `styles/tokens.css`, Michroma for titles and Barlow for labels). Waveform and separators use amber. The record-style red is used only for the on/off LED and CLIP.

```
+--------------------------------------------------------------------------+
| SLICER                          [ON o]      LCD: Sample loaded (4.21 s)  |
+--------------------------------------------------------------------------+
| SAMPLE plate                                                             |
| [Load file] [Clear]   breakbeat.wav                                      |
| +--------------------------------------------------------------------+  |
| | waveform canvas, 80 px, alternating slice bands, amber outline     |  |
| +--------------------------------------------------------------------+  |
+---------------------------+----------------------+-----------------------+
| SEQUENCER plate           | FILTER plate         | ENVELOPE + REVERB     |
|  (o) DENSITY   75 %       | [LP][HP][BP][NT]     |  (o) ATT   8 ms       |
|  (o) SLICE    180 ms      |  (o) CUTOFF 1200 Hz  |  (o) DEC  60 ms       |
|  (o) JITTER    20 ms      |  (o) Q       0.8     |  (o) SUS  60 %        |
|                           |                      |  (o) REL 120 ms       |
|  VU [##########] -12.3 dB |                      |  (o) WET  18 %        |
|  CLIP (o)                 |                      |  (o) SIZE 2.2 s       |
+---------------------------+----------------------+-----------------------+
```

Below the widest layout the three lower plates stack; the smallest supported width is 640 px. Tempo, swing, start and stop are in the page header, volume and pan are on the Console strip.

## Tests

All tests are pytest functions in `02_backend/tests/test_slicer.py`, run with `uv run pytest`. Module checks run inside the page with Playwright for Python; live checks use `?test=1` with `window.__rabbit` and its `onStep` log.

- `test_slicing`: slice count equals max(1, floor(duration / sliceMs)) for several durations and slice lengths; the remainder is not a slice.
- `test_step_density`: with density 0 no voice is scheduled over 1000 steps; with 100 a voice is scheduled on every step; at 75 the hit rate is within tolerance (seeded `Math.random` stub).
- `test_random_slice`: with a stubbed `Math.random`, the chosen slice and jitter follow the stub; with real randomness two runs differ.
- `test_jitter_clamp`: the start offset is always within [0, duration - 0.005].
- `test_envelope`: the gain automation of a voice follows A, D, S, R; with A+D longer than the note, events are in time order and have no overlap.
- `test_stop`: after `onStop()` all live voices are ramped to 0 and stopped; scheduled future voices are cancelled.
- `test_no_sample`: `onStep` with no sample does nothing and does not throw.
- `test_controls_before_load`: setting filter, Q, cutoff, wet and size before any file is loaded throws nothing and the values are applied when the sample arrives.
- `test_decode_error`: an undecodable file shows the error text and keeps the previous buffer.
- `test_state`: `getState` after `setState` returns the same object; the state is JSON round-trippable; missing keys fall back to the defaults.
- `test_graph`: the node chain matches the Audio graph section and has no master gain.
- `test_mount_unmount`: after `unmount()` steps still produce audio; mount redraws the waveform and unmount stops the VU loop.
- `test_tempo_follow`: the Slicer has no BPM input and no late-step guard; a changed `stepDuration` from the transport does not change slice length or note length.
- `test_ui_shared`: the module uses `ui/knob.js`, `ui/lcd.js`, `ui/led.js`, `ui/vu.js`, `ui/pad.js`, `fx/reverb.js` and no own copies, and contains no colour value in any notation.

## Open questions

- Should the Slicer keep its own reverb (wet and size) next to the Console reverb send? The source has its own reverb, so this spec keeps it; the Console send is an additional path.
- Should the cutoff stay linear (as in the source) or become logarithmic for a better low range? This spec keeps linear to keep the sound character.
- Should the VU and CLIP stay RMS-based as in the source, or use a peak detector? The source only has RMS, so it stays.
- Should the step grid of random slices be re-seeded or shared between tabs? The source has no shared seed, so none is specified.
