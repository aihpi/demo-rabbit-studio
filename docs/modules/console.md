# Console

## Sources

This module comes from one file: `RBBT-MixConsole1_Full.html` (RABBIT Studio Console, Class-A Magic Mix, Full 1.0, 1314 lines, 70994 bytes), analysed in `report_RBBT-MixConsole1_Full.json` (the MX report). Line 327 of the original is a base64 PNG logo and was skipped. Line references in this specification are the report's line references into the original. The original is not a step sequencer: it has no tempo and no BPM. It is a multitrack WAV mixing console with a looping timeline and a WAV mixdown.

## Purpose

The Console is the mixer of RABBIT Studio. It gives one channel strip to each of the five instruments (beat, synth, slicer, theremin, metronome) next to up to 32 strips for uploaded WAV tracks, with gain, pan, EQ, reverb send, mute and solo on each. The Console is the user interface of the mixer that `audio/mixer.js` builds (see docs/plan.md, section Audio engine and mixer): the strips, the send reverb, the master bus with fader, limiter and meter taps. On top of that it owns the WAV tracks, their timeline and the WAV mixdown, which now includes the instruments. The Console builds no strips of its own, provides no `output` node to the other instruments and never connects to `ctx.destination`; the engine hands each instrument the input of its strip. The Console is an instrument folder like the others (`01_frontend/instruments/console/`) and has no on/off switch: it is always on. The strip default gain, the master fader values and the meter ranges are owned by this specification and by docs/modules/shell.md; other specifications refer to them.

The Console is built at stage 6. Every parity row is ticked at stage 6, except the mixdown rows, which are ticked at stage 9. Mute and solo act on instruments and WAV tracks together.

## Feature parity

Source "MX" is `RBBT-MixConsole1_Full.html`. All 19 entries of the `features` array are listed, in the order of the report. One addition that the target application names and that no source has follows the table.

| Feature in the source | Source | In RABBIT Studio |
|---|---|---|
| Channel strips: 6 created at start (`NUM_TRACKS`=6, l.345), ADD TRACK tile appends up to `MAX_TRACKS`=32 (l.346, l.944-961), status message at the limit, no remove function | MX | Kept for WAV tracks: 6 at start, ADD TRACK up to 32, same limit message in English ("Maximum of 32 tracks reached."). No remove function, as in the source. The five instrument strips are added in front of the WAV tracks, always present, and do not count towards the 32. |
| Strip header: "CH 01" number (l.769), name input, maxLength 14, default "TRACK n" (l.773-774), name only stored in `t.name` and used in status messages | MX | Kept for WAV tracks, and the name is stored in the track state. Instrument strips carry the instrument title as a fixed name plate with no input. |
| LOAD WAV opens a hidden file input (`.wav,audio/wav,audio/x-wav,audio/*`, l.781), decodes with `decodeAudioData` into an AudioBuffer in memory (l.783-800), shows "LOADING...", error status on failure, a buffer loaded while playing starts only after the next start or seek | MX | Kept, with one change: the file is uploaded to the backend (`POST /api/samples`) and the track state holds its `sampleId`. The browser decodes the downloaded file (`GET /api/samples/{id}`) into an AudioBuffer as in the source. The backend accepts audio types only and at most 50 MB, as in docs/plan.md, section Backend. Loading while playing does not start the buffer until the next start or seek, as in the source. The error status reads "Could not load file (not a valid audio file)." Instrument strips have no LOAD button. |
| CLEAR (x) per strip: stops the source, drops the buffer, resets the file input and the label "(empty)" (l.801-811) | MX | Kept for WAV tracks. It also clears the `sampleId` in the state. The uploaded file stays on the backend. |
| Per-strip EQ: LO-CUT 75 Hz toggle (highpass 75 Hz, 20 Hz when off), LOW shelf 120 Hz, MID peaking (Q=1) with frequency select, HI shelf 8 kHz, each gain -15..+15 dB, default 0 (l.817-859) | MX | Kept unchanged on every strip, WAV and instrument. The Console strip supplies the EQ the target application names. |
| MID frequency dropdown: 250 Hz, 500 Hz, 1 kHz (default), 2.5 kHz, 5 kHz, 8 kHz (l.847) | MX | Kept unchanged. |
| PAN knob -1..+1, default 0, label "C" within 0.03 of centre, otherwise L/R plus percent (l.661-664, l.862-866) | MX | Moves to the Console strip as the pan of the instrument channel. Kept unchanged on WAV strips, using `ui/knob.js`. |
| REV SEND knob 0..1 (shown as 0-100%), default 0 (l.867-871) | MX | Kept on every strip. The send feeds the one shared plate reverb of the Console. |
| Vertical volume fader: range input 0..1 step 0.001, position 0.75 = 0 dB, top = +6 dB, below position 0.75 linear in dB from 0 dB down to -60 dB at position 0, position <= 0.001 = silence, default gain 0.8, dB scale at +6, 0, -20, -40, -inf (l.347-372, l.693), label with one decimal (l.656-660) | MX | Moves to the Console strip as the volume of the instrument channel. Kept unchanged on WAV strips, built with `ui/fader.js`. The taper functions `posToGain` and `gainToPos` are kept as they are. The scale marks are computed from the same constants as the CSS, so the two cannot drift apart (see Defects fixed). |
| Per-strip level meter: 14x160 canvas, peak (not RMS) from AnalyserNode time-domain data, -60..+6 dB, green, amber and red gradient, white 0 dB line, peak-hold readout that never decays until clicked, click on canvas or readout resets (l.878-890, 914-922) | MX | Changes form: the canvas meter is replaced by the shared 10-segment `ui/vu.js` of the RABBIT look, so the gradient and the white 0 dB line give way to the segment colours. The peak is still read from an AnalyserNode (fftSize 256) over -60..+6 dB, which the Console passes to `ui/vu.js` as its dB range. The peak-hold readout and the click-to-reset stay. The meter registers with the one animation loop of `audio/meters.js` when the Console is mounted and has no loop of its own. |
| M (mute) and S (solo) per strip, solo overrides mute (`audible = anySolo ? t.solo : !t.muted`), several solos allowed (l.519-525, l.906-912) | MX | Moves to the Console strip. The rule is kept unchanged and covers instrument strips and WAV strips together. |
| Reverb module PLATE REVERB SEND/RETURN: generated stereo noise IR, knobs DECAY 0.2..8 s (2.2), SIZE 0..1 (0.5, pre-delay up to 45 ms and decay exponent), LO-CUT 20..1000 Hz (150), HI-CUT 1500..18000 Hz (9000) (l.445-472, l.975-985) | MX | Kept. The generator and the rebuild move to the shared module `fx/reverb.js` (`createReverb(ctx, { decay, preDelay, power, toneHz, mix })`, generated noise IR, rebuilt 150 ms after the last change of `decay`, `preDelay` or `power`). The Console passes its own values: `decay` from DECAY, `preDelay = size * 0.045` and `power = 1.2 + (1 - size) * 2.2` from SIZE. The LO-CUT and HI-CUT filters stay in the reverb return of the mixer. The Console is the only place with these four knobs. |
| MASTER BUS: REV RTN fader (default 1.0), MASTER fader (default 1.0), same taper as strips (l.1008-1009); stereo L/R master meter (two 12x160 canvases), peak-hold text "L / R", click resets (l.1011-1042) | MX | Kept, with the master bus built by `audio/mixer.js`. The Console shows the master fader and the reverb return fader with the values of this specification. The master meter is two `ui/vu.js` columns, with the "L / R" peak-hold text and click-to-reset kept. |
| Transport: rewind, PLAY/STOP toggle (label switches between PLAY and STOP), separate STOP, STOP keeps the last UI-updated position and PLAY resumes from it (l.584-588, l.1046-1053) | MX | The PLAY/STOP toggle and the separate STOP are replaced by start and stop of the global transport in the header. The Console has no PLAY or STOP of its own. Rewind stays on the Console as a timeline control (it moves the WAV timeline to loop start, or to 0 when looping is off). Whether STOP keeps a position is an open question (see Open questions). |
| LOOP OFF/ON (default off); on: PLAY starts at `loopStart`, playback restarts from `loopStart` at `loopEnd` (l.1056-1065, l.641-644); without loop, playback ends when all sources end, status "Weergave voltooid." | MX | Kept as a timeline control with default off. The loop restarts on the audio clock instead of the main thread (see Transport behaviour and Defects fixed). The end status reads "Playback finished." |
| Timeline: 34 px bar, red playhead, amber loop region, two red draggable loop handles (minimum loop 0.05 s), click-to-seek, time display "MM:SS.s / MM:SS.s", duration = longest buffer, seeking while playing restarts all sources (l.1089-1157, l.614-619) | MX | Kept and owned by the Console. How it relates to the global transport is an open question. The playhead is written to the DOM from the rAF loop, never into state. |
| Checkbox "loop only" for mixdown (render only the loop range) and the MIXDOWN button (l.1067-1077) | MX | Kept. Labels in English ("Mix down loop selection only", "Mixdown to WAV"). The mixdown now includes every instrument that is switched on, except the Theremin. The source renders the loop range only when it is longer than 0.05 s and refuses to render when no WAV is loaded (status "Nothing to render, load WAV files first."); the second rule is open for instrument-only mixdowns (see Open questions). Ticked at stage 9. |
| Offline mixdown: whole graph rebuilt in an OfflineAudioContext (2 ch, device sample rate), length of range plus reverb tail (decay + 1.0 s), 16-bit PCM WAV, download `mixdown_YYYY-MM-DD-HH-MM-SS.wav` (l.1163-1255, l.1257-1311) | MX | Kept: same tail, same 16-bit encoder, same file name. The graph is built by `buildMixer(state, ctx)` of `audio/mixer.js`, the same function as the live mixer. The instruments reach the offline render as described under Audio graph. Ticked at stage 9. |
| Status line (amber text) under the master section with Dutch messages (l.612, l.1079) | MX | Kept as an `ui/lcd.js` line in amber, with English messages. |

One further thing that the target application names and that no source has is added to this module: the five instrument strips themselves. Project save and load is not part of this module; it belongs to the application (`core/store.js`, `core/api.js`) and stores the Console state next to the state of every instrument.

Where two sources implement the same thing differently, no rule is needed: the Console has a single source. Where the Console and the shared modules overlap, the shared module wins: `ui/knob.js` replaces the knob of the source, `ui/fader.js` replaces the rotated native range, `ui/vu.js` replaces the canvas meter, `ui/lcd.js` replaces the status line and `fx/reverb.js` replaces the inline impulse generator. The reason is the target decision that instruments use the shared modules instead of own copies.

Removed by decision: nothing in this source. The source has no wall-clock start and no countdown (the report's clockSync section confirms this), so there is nothing to remove.

## Controls

| Control | Component type | Range | Default | Source |
|---|---|---|---|---|
| Track name (WAV strips) | text input | max 14 characters | "TRACK n" | l.773 |
| LOAD WAV | button and file input | one file per strip, audio type only, at most 50 MB | empty | l.781 |
| CLEAR | button | none | none | l.801 |
| LO-CUT 75 Hz | toggle (LED) | on or off (20 Hz when off) | off | l.822-827 |
| LOW | knob | -15..+15 dB, shelf 120 Hz, shown as signed integer | 0 | l.839 |
| MID | knob | -15..+15 dB, peaking Q=1 | 0 | l.835 |
| HI | knob | -15..+15 dB, shelf 8 kHz | 0 | l.831 |
| MID frequency | select | 250, 500, 1000, 2500, 5000, 8000 Hz | 1000 | l.847 |
| PAN | knob | -1..+1 | 0 | l.862 |
| REV SEND | knob | 0..1 | 0 | l.867 |
| Volume | fader | position 0..1 (step 0.001), gain 0..+6 dB (1.995x), position 0.75 = 0 dB | gain 0.8 (about -1.9 dB) | l.347, l.894 |
| M (mute) | toggle (LED) | on or off | off | l.906 |
| S (solo) | toggle (LED) | on or off | off | l.906 |
| Strip meter | VU, 10 segments | -60..+6 dB peak | none | l.878 |
| DECAY | knob | 0.2..8 s | 2.2 | l.975 |
| SIZE | knob | 0..1 | 0.5 | l.975 |
| Reverb LO-CUT | knob | 20..1000 Hz | 150 | l.975 |
| Reverb HI-CUT | knob | 1500..18000 Hz | 9000 | l.975 |
| REV RTN | fader | silence, -60 dB to +6 dB, same taper | gain 1.0 (0 dB) | l.1008 |
| MASTER | fader | silence, -60 dB to +6 dB, same taper | gain 1.0 (0 dB) | l.1009 |
| Master meter L and R | two VU columns | -60..+6 dB peak | none | l.1011 |
| Rewind | button | to `loopStart` if looping, else 0 | none | l.1046 |
| LOOP | toggle (LED) | on or off | off | l.1056 |
| Loop handles | drag on timeline (mouse and touch) | minimum loop length 0.05 s | `loopStart` 0, `loopEnd` longest buffer | l.1089-1157 |
| Timeline seek | click on timeline | 0..longest buffer | 0 | l.1089 |
| Peak reset | click on a meter or its readout | none | none | l.914-922 |
| Loop only | checkbox | on or off | off | l.1067 |
| MIXDOWN | button | none | none | l.1067 |
| ADD TRACK | tile | up to 32 WAV tracks | 6 tracks | l.944-961 |

Knob drag: vertical (mouse or touch), full range per 140 px, double-click resets to the default, sweep -135 to +135 degrees (l.738-758). The source knobs have no keyboard access and no wheel support. These behaviours belong to `ui/knob.js` and are the reference for it; the shared knob adds keyboard and ARIA support. The Console knobs use the default reset (a double-click restores the default). The global header holds start, stop, tempo and swing; the Console has none of them.

## State

The state of the Console is one JSON object, the Console entry of the project. Instrument strips are stored by instrument id. The audio files of WAV tracks are referenced by the key `sampleId`, the id the backend returned on upload. `schemaVersion` belongs to the project, not to this object. Which view or tab is shown is interface state and never enters the project.

```json
{
  "master": { "volume": 1.0, "reverbReturn": 1.0 },
  "reverb": { "decay": 2.2, "size": 0.5, "locut": 150, "hicut": 9000 },
  "timeline": { "loop": false, "loopStart": 0, "loopEnd": 12.4, "loopOnlyMixdown": false },
  "strips": {
    "beat":      { "volume": 0.8, "pan": 0, "send": 0, "lowCut": false, "low": 0, "mid": 0, "midFreq": 1000, "high": 0, "mute": false, "solo": false },
    "synth":     { "volume": 0.8, "pan": 0, "send": 0, "lowCut": false, "low": 0, "mid": 0, "midFreq": 1000, "high": 0, "mute": false, "solo": false },
    "slicer":    { "volume": 0.8, "pan": 0, "send": 0, "lowCut": false, "low": 0, "mid": 0, "midFreq": 1000, "high": 0, "mute": false, "solo": false },
    "theremin":  { "volume": 0.8, "pan": 0, "send": 0, "lowCut": false, "low": 0, "mid": 0, "midFreq": 1000, "high": 0, "mute": false, "solo": false },
    "metronome": { "volume": 0.8, "pan": 0, "send": 0, "lowCut": false, "low": 0, "mid": 0, "midFreq": 1000, "high": 0, "mute": false, "solo": false }
  },
  "tracks": [
    { "name": "DRUMS", "sampleId": "a1f3c9", "volume": 0.8, "pan": -0.2, "send": 0.1, "lowCut": true, "low": 0, "mid": 3, "midFreq": 2500, "high": -2, "mute": false, "solo": false },
    { "name": "TRACK 2", "sampleId": null, "volume": 0.8, "pan": 0, "send": 0, "lowCut": false, "low": 0, "mid": 0, "midFreq": 1000, "high": 0, "mute": false, "solo": false }
  ]
}
```

The example shows default values, except for the first track. `volume` is the fader gain (0 to 1.995), not the fader position. `tracks` holds 6 to 32 entries. `sampleId` is null for an empty strip. A project that references an unknown `sampleId` is rejected by the backend with status 422. `loopEnd` is the longest buffer in seconds, set on first load (l.544). Peak-hold values and the playhead are not state. The source has no persistence (stateAndPersistence), so this shape is new; it follows the source globals (l.480, l.530-539) and the target rule of one plain JSON object.

## Audio graph

The graph below is built by `buildMixer(state, ctx)` in `audio/mixer.js`, not by the Console. The engine calls it once at start, before any instrument exists, with the project as `state`; the mixdown calls the same function with an `OfflineAudioContext`, so the live and the offline graph cannot diverge. One AudioContext exists for the whole application; `audio/engine.js` creates it at page load and resumes it on the first user gesture anywhere on the page. The Console is the user interface of this graph: its controls write the strip values into the project, and the Console mounts its meters on the taps of the mixer.

Per WAV track (from l.482-513):

```
AudioBufferSourceNode (created per start)
  -> BiquadFilter highpass (20 Hz, 75 Hz with LO-CUT)
  -> BiquadFilter lowshelf 120 Hz
  -> BiquadFilter peaking (Q 1, midFreq)
  -> BiquadFilter highshelf 8 kHz
  -> GainNode volume (0.8)
  -> StereoPannerNode
  -> GainNode muteGain (1 or 0, 10 ms time constant)
  -> AnalyserNode (fftSize 256)
  -> masterInput
muteGain -> GainNode sendGain (0) -> reverbInputBus
```

Per instrument strip: the engine hands the instrument the input of its strip as `output`, which is the start of the same chain, at the highpass filter. The instrument connects to `output` and to nothing else. The strip has no source node, because the instrument creates its own sources.

Reverb bus (l.439-443, l.447-467):

```
reverbInputBus (gain 1) -> ConvolverNode (generated stereo noise IR)
  -> BiquadFilter highpass (locut) -> BiquadFilter lowpass (hicut)
  -> GainNode reverbReturn (1.0) -> masterInput
```

Impulse response, generated by `fx/reverb.js` from the values the Console passes: length `decay` seconds, pre-delay `preDelay = size * 0.045` s, envelope `(1 - t)^power` with `power = 1.2 + (1 - size) * 2.2`, uniform noise on each of two channels. The impulse is rebuilt 150 ms after the last change of DECAY or SIZE. The reverb filters follow LO-CUT and HI-CUT with a 20 ms time constant.

Master (l.388-402), built by the mixer:

```
masterInput -> GainNode masterFader (1.0) -> limiter -> ctx.destination
masterFader -> ChannelSplitter(2) -> AnalyserNode L, AnalyserNode R (fftSize 512)
```

Only the mixer connects to `ctx.destination`.

Mute and solo: the strip is audible when `anySolo ? solo : !mute`. The rule covers instrument strips and WAV strips together and ignores any per-channel solo inside an instrument. Mute and solo set the gain of `muteGain` only. The reverb send is tapped after `muteGain` (post fader, pan, EQ and mute), as in the source, so a muted strip also sends nothing to the reverb.

Offline mixdown, ticked at stage 9: the render follows the mixdown of docs/plan.md.

1. Create an `OfflineAudioContext` with 2 channels, the length of the range plus decay + 1.0 s, and the sample rate `ctx.sampleRate` of the live context. A test asserts the rate, because a `ConvolverNode` buffer must match the sample rate of its context.
2. Call `buildMixer(state, offlineCtx)`, without analysers.
3. Call `createInstrument` for every instrument that is switched on (`on.<id>`), except the Theremin, with the offline context, its strip input as `output` and an offline transport from `createOfflineTransport({ bpm, swing })`. Call `setState` and wait for `ready()`.
4. Start the WAV sources of the range, call `onStart(0)` and then `onStep` for every step of the range.
5. Render, encode with `audio/wav.js` and offer the file.

The Theremin depends on a live camera and is left out; the status line says so. The Slicer picks slices at random, so its part has the same settings as what was heard, but not the same slices. The reverb is built again with the live values, and its random tail differs slightly (see Open questions).

## Transport behaviour

The Console implements `onStart`, `onStop` and `onStep` of the instrument contract in docs/plan.md for its WAV tracks. The global transport owns start, stop, tempo and swing (tempo 40 to 300 BPM, swing 0 to 70 %); see docs/plan.md, section Transport.

On start, the Console starts all loaded WAV sources at one shared `startAt = ctx.currentTime + 0.06` with the same offset (l.566), so the tracks stay aligned. `audio/engine.js` owns the one AudioContext and resumes it on the first user gesture anywhere on the page, so the Console does not resume it itself. Empty strips and strips without a decoded buffer stay silent and do not throw.

On stop, the Console cancels its scheduled sources and ends all sounding WAV sources with a short fade (rule 8 of the plan), and stops the timeline playhead. The instruments stop themselves through their own `onStop`; the Theremin is not silenced by the transport, only by its own switch. The position handling is an open question.

On each step, `onStep(stepIndex, time, stepDuration)` has no audio to schedule, because the Console has no step grid and no tempo in the source. The Console uses the call only to keep its loop restart on the audio clock: it checks whether the next loop wrap falls inside the coming step window and schedules the new sources for that exact time (see Defects fixed). The step grid does not drive any control. The Console has no guard against late steps; the transport skips a step that is more than 5 ms late.

On tempo change, nothing happens to the WAV tracks. They play in seconds and do not follow the tempo (open question).

While the tab is hidden, the audio keeps running, because every source is started on the audio clock and the loop wrap is scheduled from `onStep`, not from rAF. The rAF loops (playhead, meters) pause with the tab, which costs nothing audible. This fixes the stall of the source (l.641-644).

On seek while playing, the Console restarts all sources at the new position, as in the source (l.594), with a 10 ms gain ramp on the strips to avoid a click.

## Defects fixed

Only defects that broke behaviour are listed here. Line references are the report's.

1. Loop wrap rebuilds all sources from the main thread with a 60 ms delay (l.641-644, l.574, l.599-610): audible gap on every loop, and a stall when the tab is hidden because rAF is throttled. The loop is now scheduled on the audio clock.
2. Race in loop restart: both the rAF path and the `onended` path call `startPlaybackAt` (l.561, l.574, l.599-610), so a double restart is possible. One scheduler owns the restart.
3. The exact position is lost at stop because `seekPosition` is only updated by rAF (l.584-588, l.646-647). The position is computed from the audio clock at stop, whichever rule open question 2 selects.
4. Seeking while playing restarts all sources immediately with no fade (l.594): click. A short gain ramp is added.
5. The reverb IR is replaced live and can click (l.459 to l.467). The swap is debounced in the shared `fx/reverb.js`, which also smooths the swap.
6. Controls that throw before a file is loaded: strips with no buffer must start, mute, solo and mix down without error. The empty `catch` blocks on `source.stop` (l.552, l.561, l.802) are replaced by a check on the source.
7. A failed OfflineAudioContext construction or render for a very long mix (memory) is caught only around the render (l.1239-1254). The construction moves inside the same try block, and the status line shows an error message instead of leaving the page in "Rendering...".

Only these seven items go to `CHANGELOG.md`, each with the file and line it came from.

## Rewrite and accessibility changes

These clean-ups come with the rewrite. They did not break behaviour and are not listed in `CHANGELOG.md`.

1. Live graph and offline graph are written twice (l.482-503, l.1199-1237). One `buildMixer(state, ctx)` in `audio/mixer.js` serves both.
2. Window listeners per knob are never removed (l.752-757), about 260 with 32 tracks, each running on every mouse move. The shared `ui/knob.js` captures the pointer on its own element and adds keyboard and ARIA support. The loop handles use one pointer handler instead of the duplicated mouse and touch code (l.1134-1151).
3. One rAF loop per strip, plus master and timeline, never cancelled (l.915-922, l.1031): the one loop of `audio/meters.js` serves all meters, and the Console registers its meters on mount and deregisters them on unmount.
4. A gradient is allocated on every frame in `drawMeterBar` (l.421). The segmented `ui/vu.js` has no gradient allocation.
5. AudioContext created at load, with no rule for resuming it (l.386). `audio/engine.js` creates the one context at page load and resumes it on the first user gesture anywhere on the page.
6. Hard-coded Dutch strings (l.565, l.607, l.795, l.798, l.950, l.1071, l.1108): replaced by English.
7. Hand-kept fader geometry constants (l.374-385) must be edited in CSS and JS together. `ui/fader.js` computes the scale marks from one constant set.
8. The Google Fonts request at load (l.6) leaks the visitor's IP address and breaks offline use. The application serves Michroma and Barlow from its own folder.

Not fixed on purpose: the 16-bit hard clip of the encoder (l.1307).

## Kept on purpose

Plain Web Audio biquads, gains and a convolver, with no Class-A modelling: the name is cosmetic and stays cosmetic. The fader taper stays, including +6 dB at the top and the true silence at position 0.001 or lower. The default fader gain stays at 0.8. The master bus ends in the limiter of `audio/mixer.js`, which the source did not have; the encoder still hard-clips (l.1307) without dither. The EQ stays fixed at three bands with shelf frequencies of 120 Hz and 8 kHz and Q=1 for the mid band. The reverb stays a plain exponentially decaying white noise with a pre-delay, without early reflections, damping or modulation; the tail is random at every rebuild (l.459), so it sounds slightly different each time DECAY or SIZE is touched. The reverb send is tapped post fader, pan and mute. Meters read the time-domain peak over 256 or 512 samples with instant attack and release and a peak-hold readout that only a click resets; true peaks between frames can be missed. The mixdown keeps the device sample rate (not selectable) and 16-bit stereo output with no dither, and it has no progress indicator or cancel. Wheel support on knobs is not required of the Console. The layout stays a fixed pixel layout with horizontal scroll; the smallest supported width is 640 px, and there are no rules or tests for narrower screens.

## Layout

```
+--[RABBIT]-- header: [START][STOP]  BPM [120]  SWING [0] -------------------------+
|                                                                                    |
|  CONSOLE                                         SERIAL No. 001                    |
| +------+ +------+ +------+ +------+ +------+ +------+ +------+     +-----------+ |
| | beat | |synth | |slicer| |therem| |metro | | CH01 | | CH02 |.... | MASTER BUS| |
| | o  o | | o  o | | o  o | | o  o | | o  o | | o  o | | o  o | +ADD| DECAY SIZE| |
| |      | |      | |      | |      | |      | |LOAD x| |LOAD x|     | LO-C HI-C | |
| |[LC]  | |[LC]  | |[LC]  | |[LC]  | |[LC]  | |[LC]  | |[LC]  |     |           | |
| |HI MID| |HI MID| |HI MID| |HI MID| |HI MID| |HI MID| |HI MID|     | REV  MST  | |
| | LOW  | | LOW  | | LOW  | | LOW  | | LOW  | | LOW  | | LOW  |     | RTN       | |
| |[250..]| ...                                                       | |fader|    | |
| |PAN SND| |PAN SND| ...                                             |  L R meter| |
| | ||  | | | ||  | |                                                 | [LOOP]    | |
| | VU|fad| | VU|fad|                                                 | [x] loop  | |
| | [M][S]| | [M][S]|                                                 | [MIXDOWN] | |
| +------+ +------+ +------+ +------+ +------+ +------+ +------+     | status    | |
|                                                                      +-----------+ |
| +--------------------------------------------------------------------------------+ |
| | [<<]  |-----[loop]=====|>----playhead----|---[loop end]------  00:03.2/00:12.4 | |
| +--------------------------------------------------------------------------------+ |
+------------------------------------------------------------------------------------+
```

Colours follow the RABBIT chassis and come from `styles/tokens.css`; no colour value in any notation appears in the Console files. The accent red marks the playhead, knob pointer, active LO-CUT and mute, and loop handles; the amber marks the plate text, status line, time display and solo. Michroma for titles and buttons, Barlow for labels. The five instrument strips sit on the left, in tab order, with a divider before the WAV strips. The strip order, with each strip 136 px wide in the source, is a starting size. The strips scroll horizontally inside the rack when more than the screen width are shown. The neon colours of the older "CP5" files do not apply to the Console.

## Tests

All tests are pytest files in `02_backend/tests/` (for example `test_console.py`), run with `uv run pytest`. Checks of single functions run inside the page with Playwright for Python; live checks use the `?test=1` mode with `window.__rabbit` and its `onStep` log. Tests 11, 12 and 17 are ticked at stage 9 with the mixdown, test 15 at stage 3 with the application's project save and load, and the others at stage 6.

1. `posToGain` and `gainToPos` agree: position 0.75 gives gain 1.0, position 1 gives +6 dB (1.995x), position 0 gives silence (position 0.001 or lower gives 0), position just above 0 approaches -60 dB, and the round trip stays within 0.001.
2. The mute and solo rule: with no solo, a muted strip is silent; with one solo, only that strip is audible; with two solos, both are audible; solo overrides mute.
3. `createReverb` in `fx/reverb.js`, called with the Console mapping (`preDelay = size * 0.045`, `power = 1.2 + (1 - size) * 2.2`), builds an impulse of two channels with length `floor(decay * sampleRate) + floor(preDelay * sampleRate)`, with a leading silence of `floor(preDelay * sampleRate)` samples and a decaying envelope. A change of `decay` rebuilds it once, 150 ms after the last change.
4. `buildMixer` builds the same node chain for a WAV strip and an instrument strip, and the offline and live calls produce the same node count apart from analysers. The Console itself creates no node and never connects to `ctx.destination`.
5. LO-CUT sets the highpass to 75 Hz when on and to 20 Hz when off.
6. A strip with no buffer starts, stops, seeks and mixes down without throwing.
7. A loop wrap at `loopEnd` is scheduled on the audio clock: the next source start time equals the wrap time, with no 60 ms gap.
8. With the tab hidden (rAF paused), the loop still wraps and the sources keep playing. This is a manual check, as in docs/plan.md, section Tests.
9. A seek restarts all sources at the new position and ramps the strip gains.
10. The WAV encoder writes a 44-byte header, 16-bit PCM, interleaved stereo, and clips values outside -1..1.
11. The mixdown length equals the range plus decay + 1.0 s, the offline context has the sample rate of the live context, and with "loop only" it renders only the loop range (when longer than 0.05 s).
12. The mixdown includes every instrument that is switched on (beat, synth, slicer, metronome) and leaves out the Theremin, and the status line says that the Theremin is left out.
13. `getState` and `setState` round trip: `setState(getState())` restores every knob, fader, toggle and loop value, and the strips match the model.
14. Adding a 33rd WAV track is refused with the limit message, and tracks 7 to 32 each get a working strip.
15. A project save and load by name restores the Console state and refetches the audio files by `sampleId`. A project with an unknown `sampleId` is rejected with status 422.
16. The mid frequency select offers exactly 250, 500, 1000, 2500, 5000 and 8000 Hz.
17. A failed offline render (for example a very long mix) shows an error status and leaves the Console usable.
18. Playwright: load a WAV file, press the header start, see the playhead and the meters move, press the header stop, and download a mixdown file with the expected name pattern.

## Open questions

1. How does the looping WAV timeline relate to the global transport? The source has no tempo and no step grid; its timeline runs in seconds, and its own PLAY and STOP move it. The target application has one global transport with BPM. The sources cannot settle this. A provisional default for the first version: the timeline starts when the global transport starts and stops when it stops, the loop is in seconds and does not follow the tempo, and the WAV tracks never change speed with the BPM. Alternatives are a loop length in whole bars, or a bar-quantised start of the WAV tracks. Please decide before implementation.
2. Does global STOP keep the timeline position (the source resumes from the last position, l.584-588) or reset to `loopStart`, and does the global start resume from there?
3. Which range does a mixdown take when the loop is off and the instruments run on a step grid: the longest buffer, a fixed number of bars, or the time between start and stop? The source refuses to render when no WAV is loaded and takes its length from the longest buffer, so the sources also give no rule for an instrument-only mixdown.
4. Whether the WAV strips can be reordered or removed: the source has no remove function, so the Console keeps none, but the report lists this as a weakness without a decision.
5. Whether the five instrument strips are visible when their instrument is switched off, or hidden until it is switched on: the sources do not say.
6. Whether the project state stores the reverb impulse seed so that a loaded project sounds identical (the source uses `Math.random` at every rebuild, l.459). Default here: it does not, and the tail differs slightly after every load.
