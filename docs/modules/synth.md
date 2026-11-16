# Synth

## Sources

The module "Synth" (id `synth`) is built from two original files, read through their static analysis reports.

| Short name | Original file | Report |
|---|---|---|
| SM | RABBIT-SynthMachine1_Full.html ("RABBIT Synth Machine", 16-step polysynth piano roll sequencer 1.0, 1240 lines) | report_RABBIT-SynthMachine1_Full.json |
| S3 | CP5_TT_20251118_SynthSEQ3.html ("Tom's Bass Station", monophonic 16-step bass sequencer, 585 lines) | report_CP5_TT_20251118_SynthSEQ3.json |

SM is the base. S3 adds a second sequencer layer, the bass view, which keeps its scale degrees, its scales, its octave range and its own voice settings. Line numbers in this document refer to the original files as quoted in the reports.

## Purpose

Synth is one instrument tab with two sequencer views on one 16-step grid. The ROLL view is the SM polysynth: 36 pitch rows, up to 16 voices, HPF, LPF, reverb and delay. The BASS view is the S3 bass sequencer: one note per step, chosen as a scale degree, with its own lowpass filter and envelope. Both patterns play together whenever the instrument is switched on, and both play through one output, the Synth channel strip that `audio/mixer.js` builds and the Console shows. The engine hands the strip input to the instrument as `output`.

## Feature parity

Where SM and S3 implement the same thing differently, the table names the winner and the reason. "Header" means the global transport in the page header. "Console" means the user interface of the channel strip of this instrument; the strip itself comes from `audio/mixer.js`. "Removed by decision" means the decided target application removes it.

| Feature in the source | Source | In RABBIT Studio |
|---|---|---|
| Piano roll: 16 steps x 36 pitch rows, MIDI 48 (C3) to 83 (B5), row height 16 px, key column 46 px, roll canvas 576 px tall (36 x 16) with ruler (20 px) and velocity lane (76 px), scroller capped at 400 px, initial scrollTop 5 rows | SM | Kept as the ROLL view. Canvas drawing stays. Colours are read once at mount from the CSS custom properties of the RABBIT look instead of hex literals in the draw code. |
| Demo pattern loaded at start: 4 chords x 3 notes (length 4, vel 84; A-C-E at step 1, F-A-C at 5, G-C-E at 9, G-B-D at 13) plus 9 melody notes (vel 70-116), 21 notes in all, info LCD "DEMO PATTERN" | SM | Kept as the default state of a new project. |
| Add note by double-click on an empty cell (350 ms, same column and pitch), len 1, vel 100, then resize mode | SM | Kept. Hand-rolled double-click detection stays, so a drag after the second click lengthens the note. |
| Delete note by double-click on a note, or Delete / Backspace on the selected note | SM | Kept. |
| Select note by single click; drag body moves in time and pitch; drag right edge resizes (hotspot clamp(0.3*colW, 5, 9) px); length 1..(16 - start); moving does not check collisions | SM | Kept, including the missing collision check (see Kept on purpose). The store is written on pointerup only. |
| Keyboard on the focused roll, selected note only: Up/Down pitch +/-1 (Shift +/-12) clamped to 48..83, Left/Right start +/-1 clamped to 0..16-len, each pitch change previews | SM | Kept. |
| Piano-key strip (x < 46 px): click previews the pitch for 0.22 s at vel 100, key flashes 180 ms, hover highlights the row and key and shows note name and step in the info LCD | SM | Kept. The preview plays through the instrument output, also while the transport is stopped. |
| Velocity lane: click or drag vertically sets velocity 1..127; target is the selected note if it starts in that column, else every note starting in that column; stems with round heads are drawn per note | SM | Kept. |
| RND button (A-minor pentatonic pool, offsets 0,3,5,7,10 from A; per group of 4 steps 75 % chance of a root plus fifth, length 4, vel 70-94 and 64-83; melody pass 55 % per step, 30 % length 2, vel 60-126; replaces the pattern) and CLR button | SM | Kept as the ROLL view buttons. They act on the ROLL pattern only. |
| Transport Start and Stop (Start is a play triangle, Stop a square and disabled at first), with the 4 s wall-clock lock, the "LOCKING..." status with blinking LED, the "STARTS IN x.xs" countdown and the "LOCKED @" text | SM | Start and Stop move to the header. The wall-clock start, the countdown, the LED blink and the lock texts are removed by decision. |
| Playback: 16th-note steps, stepDur = 60/bpm/4, 16-step loop (240/bpm s, 2.000 s at 120 BPM), playing step shown on ruler, column tint and note glow | SM | The step clock moves to the header transport, which calls `onStep`. The ROLL view keeps the visual step display. |
| Swing 0..70 %: even-indexed steps lengthened, odd-indexed shortened by stepDur*swing/100*0.5 (maximum shift 0.35 of a step) | SM | Moves to the header. The transport uses the SM formula and passes the already swung `time` to `onStep`. Note durations use the nominal `stepDuration`, as SM does. |
| Tempo knob 40..300 BPM, default 120, reset 120, with +/- buttons (+/-1, Shift +/-5), swing knob and volume knob (-30..+6 dB, step 0.5, default -4 dB, reset -4) | SM | Tempo and swing move to the header, volume to the Console. None of them is on the Synth tab. |
| Polyphony cap 16, counted at schedule time as voices whose [start,end) contains the new start, extra notes silently dropped, "n/16" LCD | SM | Kept. Bass voices count towards the cap. |
| Oscillator waveform: sine, sawtooth, triangle, square (default sawtooth), level factor (sine 1, tri 0.9, saw 0.55, square 0.5), click previews A4 (MIDI 69) at vel 96 | SM | Kept as the ROLL wave. SM and S3 both offer sine, square and sawtooth; SM wins because it has the extra triangle and the level factors. The bass wave (three buttons) is a separate control, see the next rows. |
| Octave shift -2..+2 (default 0) transposes playback and preview by 12*octave and relabels the roll key names; stored pitches do not change | SM | Kept for the ROLL view. It does not affect the bass view. |
| Detune -50..+50 cents on the single oscillator per voice, read when a voice is created | SM | Kept for ROLL voices. |
| ADSR amplitude envelope with an envelope-shape canvas preview (84 px high canvas, quadratic curves) | SM | Kept for ROLL voices. One drawing routine serves the preview. |
| Two filters in series (HPF then LPF), each with cutoff and resonance | SM | Kept for ROLL voices. |
| Reverb (convolver with synthetic noise impulse response; decay, size, tone, mix) and tempo-synced delay (10 divisions, feedback, tone, mix) | SM | Kept. They are built from the shared modules `fx/reverb.js` (`createReverb(ctx, { decay, preDelay, power, toneHz, mix })`, with the values of SM) and `fx/delay.js` (`createDelay(ctx, transport, { division, feedback, toneHz, mix })`). The impulse response is rebuilt 150 ms after the last change of decay, pre-delay or power. The delay retimes itself through `transport.subscribe`. Bass voices also feed both effects. |
| Master volume -30..+6 dB, compressor, stereo L/R VU meters of 10 segments | SM | Master volume moves to the Console strip (volume). The compressor stays at the end of the instrument chain. The Console strip default gain and the master fader are owned by shell.md and console.md. The VU meter uses `ui/vu.js` and is fed from the compressor output. The default of -4 dB stays as a fixed trim of gain 0.631 ahead of the compressor, so the compressor sees the original levels. |
| Knob interaction: vertical drag (150 px = full range), touch drag, wheel (Shift x5), arrow keys (Shift x5), Home/End, double-click resets to a per-knob reset value where one is given, delay time shows live milliseconds | SM | Moves to the shared `ui/knob.js`. A knob with a reset value in the SM controls table keeps it. A knob without one gets `reset: null`, so a double-click does nothing; the unset option would restore the default. |
| No MIDI keyboard, no computer-keyboard piano, no pattern length control, no undo, no default-velocity control (defVel fixed at 100) | SM | Stays absent. `defVel` stays the constant 100. |
| 16-step sequencer, every step a sixteenth note, 16 steps = 1 bar of 4/4, loop forever | S3 | Same as SM: 16 steps, driven by the header transport. The BASS view uses the same step counter as the ROLL view (`stepIndex % 16`). |
| Per step: a note as scale degree 0..8 or rest (option "- rust -"), and a velocity slider 0..1, step 0.01, default 0.70, 80 px tall; default state all 16 steps rest | S3 | Kept as the BASS view. The rest option is labelled "REST" (English interface). Velocity becomes the compact vertical size of `ui/fader.js`. The default is 16 rests, so a new bass layer is silent. |
| Start button "Start (sync op x4s)" creates the AudioContext on the first click, waits for seconds % 4 == 0, shows "Wachten op sync" and "Loopt @ BPM . synced" | S3 | Start moves to the header. The 4 s wait and its status texts are removed by decision. The single AudioContext is created by `audio/engine.js`. |
| Stop button clears the scheduler timeout and sets the status "Gestopt"; it does not silence scheduled notes | S3 | Stop moves to the header. The instrument additionally silences its sounding voices in `onStop` (see Defects fixed). |
| Randomise button: 75 % chance per step, degree = floor(random*min(9, scaleLen+2)), velocity 0.35 + random*0.6 rounded to 2 decimals (0.35..0.95) | S3 | Kept as the BASS view button. It acts on the BASS pattern only. |
| Clear button: all degrees null, all velocities 0.7 | S3 | Kept as the BASS view button. |
| Four scales: Natural Minor [0,2,3,5,7,8,10] (default), Major [0,2,4,5,7,9,11], Pentatonic Minor [0,3,5,7,10], Chromatic [0..11]; root always C; a degree beyond the scale length is clamped to the last degree | S3 | Kept as a bass-only control. SM has no scales. The clamping stays (see Kept on purpose). |
| Octave selector 1..5 sets the root, midi = 12 + 12*octave (C1 about 32.7 Hz to C5 about 523.3 Hz), default 2 (C2, about 65.4 Hz) | S3 | Kept as a bass-only control ("BASS OCT"). It is separate from the ROLL octave shift (-2..+2), because the two do different things: it sets the bass root, the SM one transposes the roll. |
| Waveform choice sine, square, sawtooth (default sawtooth), read at schedule time | S3 | Kept as the bass wave with the same three options. The ROLL wave is a separate control. Both read at note start, as in both sources. |
| Global lowpass filter with live cutoff and Q sliders, one filter instance shared by all bass notes, no filter envelope | S3 | Kept as the bass lowpass, one `BiquadFilterNode` shared by all bass notes. It is separate from the ROLL HPF and LPF, so the bass keeps its own tone. Sliders become knobs. |
| ADSR amplitude envelope applied per note, values read when each note is scheduled | S3 | Kept as the bass ADSR with the S3 ranges in milliseconds. The envelope maths of SM replace the S3 maths, because the S3 maths has a defect (see Defects fixed). |
| Master volume slider with live gain update (0..1, step 0.001, default 0.8) | S3 | Moves to the Console strip (volume). The bass sound goes through the same fixed trim and compressor as the roll. |
| VU meter: one div bar following the RMS of the master output (scaled by 140, clamped to 0..100 %) | S3 | SM wins: the 10-segment stereo VU of `ui/vu.js` replaces the bar. The synth passes the dB range owned by shell.md and console.md. One meter serves both views. |
| Note length = max(0.03 s, 0.9 * step duration), step duration 60/BPM/4 (0.375 s at 40 BPM, 0.125 s at 120 BPM, about 0.068 s at 220 BPM) | S3 | Kept for bass notes. ROLL notes keep the SM length n.len*stepDur*0.96. The two factors are different sound characters and both stay. |
| Tempo slider 40..220 BPM, default 120, changeable while running; effective from the next step with no phase correction | S3 | Moves to the header (tempo, 40..300 BPM, from SM). The transport reads tempo at every step, as both sources do. |
| No step playhead indicator; `.active` only marks steps that hold a note | S3 | Fixed by adding a step LED per step in the BASS view, driven by `onStep` like the ROLL playhead. The `.active` outline for "step has a note" stays. |
| Footer tip explaining the x4 s sync; no other features (no save/load, export, recording, MIDI, shortcuts, multiple tracks) | S3 | The tip is removed by decision (the 4 s sync is removed). Save and load happen through the project backend. Export, recording and MIDI stay absent. |

## Controls

The tempo, swing and volume controls are not on the Synth tab (header and Console). All ranges and defaults come from the reports. "Reset" is the value a double-click restores.

| Control | Component type | Range | Default | Source |
|---|---|---|---|---|
| Wave (ROLL) | 4 toggle buttons (`.btn--toggle`: sine, saw, tri, sqr) with SVG wave icons, 2x2 grid | 4 options | sawtooth | SM |
| Octave (ROLL) | stepper (- / +) with LCD | -2..+2 | 0 | SM |
| Detune | knob, step 1 | -50..+50 ct | 0 (reset 0) | SM |
| LPF Cut | knob | 0..100, mapped 20*1000^(v/100) Hz (20 Hz..20 kHz) | 78 (about 4.4 kHz), reset 100 | SM |
| LPF Res | knob | 0..100, mapped to Q 0.7..18 | 22 (Q about 4.5), reset 0 | SM |
| HPF Cut | knob | 0..100, same mapping as LPF Cut | 8 (about 35 Hz), reset 0 | SM |
| HPF Res | knob | 0..100, mapped to Q 0.7..18 | 0 (Q 0.7), reset 0 | SM |
| Attack | knob, step 1 | 0..100, mapped 0.001*2000^(v/100) s (1 ms..2 s) | 10 (about 2 ms), `reset: null` | SM |
| Decay | knob, step 1 | 0..100, mapped 0.005*600^(v/100) s (5 ms..3 s) | 62 (about 264 ms), `reset: null` | SM |
| Sustain | knob, step 1 | 0..100 %, level 0..1 | 55 %, `reset: null` | SM |
| Release | knob, step 1 | 0..100, mapped 0.01*500^(v/100) s (10 ms..5 s) | 60 (about 416 ms), `reset: null` | SM |
| Envelope scope | canvas, read-only, 84 px high | envelope shape | not applicable | SM |
| Reverb Decay | knob, step 0.05 | 0.2..6 s | 2.2 s (reset 2.2) | SM |
| Reverb Size | knob | 0..100 | 55 (`reset: null`) | SM |
| Reverb Tone | knob | 0..100, mapped 300*(16000/300)^(v/100) Hz (300 Hz..16 kHz) | 55 (about 2.7 kHz), `reset: null` | SM |
| Reverb Mix | knob | 0..100 % | 28 % (reset 0) | SM |
| Delay Time | select (native select styled as fader label) | 10 options: 1/32, 1/16 T, 1/16, 1/8 T, 1/16 ., 1/8, 1/8 ., 1/4, 1/4 ., 1/2 (0.125..2.0 beat) | 1/8 . (0.75 beat, 375 ms at 120 BPM) | SM |
| Delay Feedback | knob, step 1 | 0..95 %, gain v/100*0.95 (maximum 0.9025) | 35 % | SM |
| Delay Tone | knob | 0..100 | 45 (about 1.8 kHz) | SM |
| Delay Mix | knob | 0..100 % | 18 % (reset 0) | SM |
| RND / CLR (ROLL) | 2 pill buttons (`.btn--pill`) | action | not applicable | SM |
| Voices LCD | LCD, read-only | "n/16" | "0/16" | SM |
| Roll info LCD | LCD, read-only, 230 px wide | text | "DEMO PATTERN" | SM |
| Output VU | 2 x 10-segment VU | L and R | not applicable | SM |
| View switch (ROLL / BASS) | 2 toggle buttons (`.btn--toggle`) | 2 options | ROLL | none of the sources (interface state only, needed to hold two views in one tab; it never enters the project) |
| Bass wave | 3 toggle buttons (`.btn--toggle`: sine, square, saw) | 3 options | sawtooth | S3 |
| Bass scale | select | Natural Minor, Major, Pentatonic Minor, Chromatic | Natural Minor | S3 |
| Bass octave | knob, step 1 | 1..5 | 2 | S3 |
| Bass cutoff | knob, linear | 60..8000 Hz, step 1 | 900 Hz | S3 |
| Bass resonance | knob | 0.1..20, step 0.1 | 1.0 | S3 |
| Bass Attack | knob | 0..1000 ms, step 1 | 10 ms | S3 |
| Bass Decay | knob | 0..2000 ms, step 1 | 120 ms | S3 |
| Bass Sustain | knob | 0..1, step 0.01 | 0.60 | S3 |
| Bass Release | knob | 0..2000 ms, step 1 | 180 ms | S3 |
| Bass RND / CLR | 2 pill buttons (`.btn--pill`) | action | not applicable | S3 |
| Step note (x16) | select | REST or degree 0..8 | REST | S3 |
| Step velocity (x16) | compact vertical `ui/fader.js`, 80 px tall | 0..1, step 0.01 | 0.70 | S3 |
| Step LED (x16) | LED | on while the step plays | off | none (fix, replaces the missing playhead of S3) |

## State

State is one plain JSON object. Tempo, swing, volume, pan and mute are not part of it. The on/off switch is stored in the project as `on.synth`, not here. `schemaVersion` is the only version key and belongs to the project, so this object has none. The ROLL/BASS view and the scroller position are interface state and never enter the project. `notes` holds ROLL notes: pitch is MIDI 48..83, start 0..15, len 1..(16 - start), vel 1..127. `bass.steps` always holds 16 entries; `degree` is `null` (rest) or 0..8. Knob values are stored as the knob values of the controls table (for example `lpf.cut` is 0..100, not Hz).

```json
{
  "roll": {
    "wave": "sawtooth",
    "octave": 0,
    "detune": 0,
    "env": { "a": 10, "d": 62, "s": 55, "r": 60 },
    "lpf": { "cut": 78, "res": 22 },
    "hpf": { "cut": 8, "res": 0 },
    "notes": [
      { "id": 1, "pitch": 57, "start": 0, "len": 4, "vel": 84 },
      { "id": 2, "pitch": 60, "start": 0, "len": 4, "vel": 84 },
      { "id": 3, "pitch": 64, "start": 0, "len": 4, "vel": 84 }
    ]
  },
  "reverb": { "decay": 2.2, "size": 55, "tone": 55, "mix": 28 },
  "delay": { "time": "1/8 .", "feedback": 35, "tone": 45, "mix": 18 },
  "bass": {
    "wave": "sawtooth",
    "scale": "minor",
    "octave": 2,
    "cutoff": 900,
    "q": 1.0,
    "env": { "a": 10, "d": 120, "s": 0.6, "r": 180 },
    "steps": [
      { "degree": 0, "velocity": 0.9 },
      { "degree": null, "velocity": 0.7 },
      { "degree": 3, "velocity": 0.7 }
    ]
  }
}
```

The example shows 3 of the 21 demo notes and 3 of the 16 bass steps; a real state holds all of them. A new instrument starts with all 21 demo notes and 16 bass rests. `scale` is one of `minor`, `major`, `pentatonic`, `chromatic`. `setState` ignores unknown keys (including an old `version` or `view`), fills missing keys with the defaults above, clamps values to the ranges, and redraws the canvases if the instrument is mounted. The `reverb` and `delay` entries keep the knob values; the instrument converts them to the parameters of `createReverb` and `createDelay`.

## Audio graph

All nodes belong to the `AudioContext` of `audio/engine.js`, live or offline (the mixdown passes an `OfflineAudioContext`). The graph is built in `createInstrument` and does not depend on mounted DOM.

Per ROLL voice (one oscillator per voice, as in the SM code; the SM comment about two oscillators is wrong):

```
OscillatorNode (type = roll.wave, frequency = mtof(midi + 12*octave), detune = roll.detune ct)
  -> GainNode (ADSR) -> synthBus
```

Per BASS voice (one oscillator and one gain per non-rest step):

```
OscillatorNode (type = bass.wave, frequency from scale degree and bass octave)
  -> GainNode (ADSR, bass envelope) -> bassFilter (shared lowpass) -> postFilter
```

Shared chain:

```
synthBus (gain 1) -> hpF (highpass) -> lpF (lowpass) -> postFilter (gain 1)

postFilter -> trim (gain 0.631, fixed) -> comp (threshold -10 dB, knee 6, ratio 8, attack 3 ms, release 0.2 s) -> output
postFilter -> reverbSend (gain mix/100) -> ConvolverNode (noise IR) -> reverbTone (lowpass) -> trim
postFilter -> delaySend -> DelayNode (max 4 s) -> delayTone (lowpass) -> trim
delayTone -> delayFb (gain) -> DelayNode
comp -> ChannelSplitter(2) -> AnalyserNode L, AnalyserNode R (fftSize 256) -> ui/vu.js
```

`output` is the input of the channel strip from `audio/mixer.js` (gain, pan, EQ, reverb send, mute, solo), then the master bus. The instrument never connects to `ctx.destination`. Both effects are fed after the filters and return before the compressor, as in SM. The dry path is mono; stereo comes from the stereo reverb impulse response. Parameter changes use `setTargetAtTime` with tau 0.015 s. The ROLL envelope is: set 0, linear ramp to peak over A, `setTargetAtTime` to sustain with tau D/3, `setValueAtTime` to the computed level at note-off, then `setTargetAtTime(0)` with tau R/4. Peak is vel/127*0.26*waveLevel. The oscillator stops at time + dur + 1.4*R + 0.05. The bass voice uses the same envelope maths with the bass A, D, S and R in seconds; its peak is vel (0..1) times a bass level, value TBD (see Open questions). Bass note length is max(0.03 s, 0.9*stepDuration). The filter parameters `bassFilter.frequency` and `.Q` are set only with `setTargetAtTime` (tau 0.015 s), never by direct `.value` writes. Voices are tracked in a set and disconnected on their `ended` event.

## Transport behaviour

Start (`onStart(time)`): resets the ring buffer and the voice list. No timer is created in the module; the transport (docs/plan.md, section Transport) owns the one scheduler. There is no wall-clock wait and no countdown. There is no late-step guard in the module; the transport skips a step that is more than 5 ms late.

Each step (`onStep(stepIndex, time, stepDuration)`): the module computes `step = stepIndex % 16`. For every ROLL note with `n.start === step` it starts a voice at `time` with duration n.len*stepDuration*0.96, unless 16 voices are already sounding at `time` (then the note is dropped silently). For the BASS step `step` with a non-null degree it starts one voice at `time` with duration max(0.03, 0.9*stepDuration). The frequency is 440*2^((midi-69)/12), with midi = 12 + 12*bass.octave + scale[min(degree, scale.length - 1)]. `onStep` does audio only. It writes `{ stepIndex, time }` into a ring buffer; a mounted interface reads it in the shared animation loop (`audio/meters.js`) when `ctx.currentTime >= time` and drops entries older than one second. Note edits are picked up on the next step. Swing is already in `time`.

Stop (`onStop`): cancels the voices that are scheduled and ends every sounding voice with a short fade and `stop()` (rule 8; the fade length is an open question). Reverb and delay tails may ring out. The step LEDs and the roll playhead clear.

Switch: the instrument switch is stored in the project as `on.synth` and is not part of the instrument state. Switching off calls `onStop`. Switching on while the transport runs calls `onStart(time)`, and the instrument plays from the next `onStep` with the current `stepIndex`. The Console has no switch for it.

Tempo change: the transport reads the tempo at every step, so the next step uses the new step duration and nothing is corrected. `fx/delay.js` retimes itself through `transport.subscribe`, so the delay time (multiplier x 60/BPM seconds) follows the tempo at once and the LCD shows live milliseconds. The synth does not retime it.

Tab hidden: audio does not depend on the DOM. `onStep` keeps scheduling from the transport lookahead, which the transport runs independently of the visible tab. If the instrument is unmounted, `mount`-time objects (canvases, LEDs) are absent, nothing reads the ring buffer, and `onStep` does no interface work. While a tab is hidden, rAF stops; entries older than one second are dropped, so the buffer cannot grow, and the next `mount` draws the current step at once. `ui/vu.js` registers with the shared loop on `mount` and deregisters on `unmount`. `getState` and `setState` work while unmounted.

## Defects fixed

Line numbers are those of the report.

| Defect | Source and lines | Fix |
|---|---|---|
| Stop does not silence anything; voices ring on up to dur + 1.4*R + 0.05 s (about 7 s at maximum release); voice gain nodes are never disconnected | SM 1213-1220, 619 | `onStop` cancels scheduled voices and fades and stops all sounding ones; voices are disconnected on `ended`. |
| Stop does not cancel notes already scheduled; oscillators are not kept in a list | S3 481-514, 553-558 | Same fix, with a voice list. |
| Double Start creates two scheduler chains and Stop clears only one; the start `setTimeout` is not stored, so Stop during the sync wait does nothing; `isRunning` is write-only | S3 432, 442-448, 544-558 | No scheduler and no wait in the module; the single transport scheduler is the only clock. |
| Start only aligned when seconds % 4 == 0 (it starts up to 999 ms off the boundary while the status says "synced"); wall-clock and audio clock never related, no drift correction, `Date.now()` not monotonic; resume not awaited; "LOCKED @" shows the planned boundary, not a measured start; the UI stays on "LOCKING" in a hidden tab | S3 530-550; SM 598, 1192-1204 | Removed with the wall-clock start. |
| setInterval or setTimeout scheduler stops in hidden tabs; late tick makes past steps play as a burst | SM 469, 1211; S3 434-435, 447 | The module has no timer. The one late-step guard (skip a step more than 5 ms late) is in the transport; the module has none. |
| rAF pause leaves `visualQueue` growing without bound and `voices` pruned only in the UI loop | SM 1175, 1226, 1230 | Ring buffer that drops entries older than one second; voices are pruned on the `ended` event. |
| Reverb impulse response rebuilt with `Math.random` on every knob tick, allocating up to 6 s x 2 channels on the main thread, can click | SM 834, 836, 545-555 | `fx/reverb.js` rebuilds 150 ms after the last change. |
| Envelope gate ignores A and D (`offTime = t0 + Math.min(noteDur, A + D + noteDur)` is always t0 + noteDur), so `setValueAtTime(vel*S, offTime)` can land inside the attack/decay ramps and click | S3 509-510 | The bass uses the SM envelope maths, which computes the level at note-off analytically. |
| Filter `setTargetAtTime` every note conflicts with slider `.value` writes; master volume `.value` writes cause zipper noise | S3 499-500, 568-573, 566 | All filter changes use `setTargetAtTime`. Volume moves to the Console. |
| Meter loop leak: `startMeter()` starts another rAF loop on every Start click | S3 411-428, 579-581 | One shared loop in `audio/meters.js`; `ui/vu.js` registers on `mount` and deregisters on `unmount` and has no loop of its own. |
| Tempo listener overwrites the status with "Loopt @ ... BPM" even when stopped | S3 314, 561-564 | Status text no longer exists; tempo is in the header. |
| No playhead, so the user cannot see which step plays | S3 144, 356, 373 | Step LED per step in the BASS view. |
| No `AudioContext` handling (no `resume()`, no try/catch); `ctx.resume()` promise ignored; `setPointerCapture` unguarded (a throw ends the gesture) | SM 560, 598, 1044, 1119; S3 394 | `audio/engine.js` owns the context and resumes it on the first user gesture on the page; the module receives `ctx`. Roll gestures wrap `setPointerCapture` in try/catch. |
| No `touchcancel` handler, so a cancelled touch leaves dragging state; wheel `preventDefault` blocks page scroll over any knob | SM 693-694, 703-706 | Both live in the shared `ui/knob.js`, which handles `touchcancel` and releases listeners. |

## Rewrite and accessibility changes

These clean-ups come with the rewrite. None of them fixes a defect that broke behaviour, so none goes to `CHANGELOG.md`. Line numbers are those of the report.

| Change | Source and lines | Result |
|---|---|---|
| `amp.gain.cancelScheduledValues(t0)` on a new GainNode does nothing | S3 503 | Call dropped. |
| 19 near-identical `makeKnob` calls, duplicated envelope path code, accent colour hard-coded in CSS, JS constant and canvas literals | SM 737-859, 814-826, 473, 820, 939, 948 | One table-driven control definition, one drawing routine for the envelope, colours read from CSS custom properties. |
| Duplicated `getElementById` lines and 9 readout listeners; hard-coded option lists 0..8 | S3 282-322, 338-348 | Generated from the controls table. |
| State held three times (DOM, `stepData`, module variables); knob value duplicated in closure and `S` | S3 327, 353-358, 363-385; SM 663, 729-733 | One state object; `getState` and `setState` are the only access. |
| Dead code: `.vel-new`, `.knob-dial.small`, `.fader-label`/`.fader-value` styles for missing faders, `S.defVel` without control, `drag.moved` never read, unused CSS variables | SM 270-271, 167-168, 512, 1032, 1071; S3 16, 19-20, 163-164 | Not carried over. `defVel` stays as the constant 100. |
| Markup not well-formed (no `<head>` close, no `<body>`) | SM 311-313, 1240 | The module mounts into an element given by the app. |
| Non-standard vertical sliders (`writing-mode: bt-lr`, Chromium and Safari only); Firefox track has no fill | S3 140-143, 91-95 | `ui/fader.js` replaces them. |
| Layout does not adapt to narrow windows (no media queries, 16 fixed step columns) | S3 54, 128 | The smallest supported width is 640 px. The BASS view has no rule and no test for narrower screens. |
| Notes can only be created and selected by pointer, the velocity lane is pointer-only, knobs have no `aria-valuetext`, step selects and sliders have no labels | SM, S3 | `ui/knob.js` and `ui/fader.js` carry keyboard and ARIA support; every step select and fader gets a label. The roll keeps its keyboard keys for the selected note. |

## Kept on purpose

- Biquad Q is mapped as if linear (0.7..18 for the roll filters, 0.1..20 for the bass filter). The Web Audio Q of a lowpass or highpass is in dB, so Res 100 % gives a strong resonance peak. This is part of the sound.
- ROLL note duration factor 0.96 and bass note duration factor 0.9 (minimum 0.03 s). They differ on purpose.
- Notes beyond the voice cap are dropped without stealing (SM 605).
- Overlapping notes of the same pitch stack and double the level, because moving does not check collisions (SM 1068-1072).
- A bass degree beyond the scale length is clamped to the last degree, not wrapped. In Pentatonic, degrees 4..8 all play the same note; in Natural Minor and Major, degrees 6, 7 and 8 do.
- The bass cutoff mapping is linear (60..8000 Hz), as in S3.
- Detune and waveform apply only to voices created after the change.
- The dry signal is mono; the reverb returns stereo.
- Reverb and delay tails run on after Stop.
- The RND algorithms of both sources (the roll pool and chances, the bass 75 % chance and degree range) stay as written.
- Dropdown degrees show numbers, not note names.
- SM envelope curve shape (`setTargetAtTime` with tau D/3 and R/4) and SM wave level factors.
- The fixed trim of gain 0.631 (the SM default volume of -4 dB) ahead of the compressor.
- The roll redraws all three canvases on every drag move and step (SM 987); this is harmless at 16 x 36 cells.

## Layout

The RABBIT look: the page, red and amber colours and the fonts Michroma and Barlow all come from `styles/tokens.css` (no colour value is written out here or in the module), plates with a vertical rotated name rail and a status LED. The Master plate of SM is gone (volume is on the Console, tempo and swing in the header); the VU meter and the voices LCD move into the sequencer plate header. The two legend lines of SM ("THE SYNTH SYNDICATE - CASE NO. 16" and "16 VOICE POLY - 16 STEP - ADSR - LPF/HPF - RVB + DLY") stay as the footer of the tab.

```
+------------------------------------------------------------------------------+
| [=== red hazard strip ===]                                                   |
| (o)                                                                    (o)   |
|  +--+-----------------------------------------------------------+            |
|  |S |  SEQUENCER   [ ROLL | BASS ]   info LCD   voices 0/16   VU|            |
|  |E |  ROLL view:                                                            |
|  |Q |  ruler  |1 . . . |5 . . . |9 . . . |13 . . .|                          |
|  |  |  keys   +------------------------------------+                        |
|  |  |  C5     |  piano roll canvas, 36 rows x 16   | scroller 400 px        |
|  |  |  ...    +------------------------------------+                        |
|  |  |  velocity lane canvas (76 px)         [RND] [CLR]                      |
|  |  |  BASS view (replaces the canvases):                                    |
|  |  |  step  1    2    3   ...  16     LED per step                          |
|  |  |  note  [0v] [REST] [3v] ...      select per step                       |
|  |  |  vel   |||  |||   |||  ...       vertical fader per step               |
|  |  |                                       [RND] [CLR]                      |
|  +--+------------------------------------------------------------+          |
|  +--+----------------------------+  +--+---------------------------+         |
|  |O |  OSC  [sin][saw][tri][sqr] |  |B |  BASS VOICE  (BASS view)  |         |
|  |S |  OCT [-]0[+]   DETUNE (k)  |  |A |  [sin][sqr][saw] SCALE [v]|         |
|  |C |                            |  |S |  BASS OCT (k) CUT (k) Q (k)|         |
|  +--+----------------------------+  |S |  A (k) D (k) S (k) R (k)  |         |
|  +--+----------------------------+  +--+---------------------------+         |
|  |F |  FILTER  HPF cut res       |  +--+---------------------------+         |
|  |L |          LPF cut res       |  |E |  ENVELOPE  A D S R  scope |         |
|  +--+----------------------------+  +--+---------------------------+         |
|  +--+----------------------------+  +--+---------------------------+         |
|  |R |  REVERB decay size tone mix|  |D |  DELAY time fb tone mix   |         |
|  |  +----------------------------+  +--+---------------------------+         |
|   THE SYNTH SYNDICATE - CASE NO. 16                                          |
+------------------------------------------------------------------------------+
```

The BASS VOICE plate is visible in the BASS view; the OSC, FILTER and ENVELOPE plates are visible in the ROLL view; REVERB and DELAY are always visible. Below 1000 px the racks become one column. The smallest supported width is 640 px; there is no rule for narrower screens.

## Tests

All tests are pytest files in `02_backend/tests/` (for this module `test_synth.py`), run with `uv run pytest`. Module-level checks (scale table, degree to MIDI, clamping) import the real module inside the page with Playwright for Python. Live checks use the `?test=1` mode with `window.__rabbit` and its `onStep` log. Offline checks render in an `OfflineAudioContext`. There is no Vitest, no `node --test` and no `*.test.js` file.

- Scale table: each of the four scales returns the offsets of the S3 list, and the Chromatic scale has 12 entries.
- Degree to MIDI: degree 0 at bass octave 2 is MIDI 36 (C2, about 65.4 Hz); a degree beyond the scale length is clamped to the last scale degree.
- Bass octave range: octave 1 gives MIDI 24 (C1) and octave 5 gives MIDI 72 (C5).
- Bass rests: a step with degree `null` starts no voice.
- Bass note length: the duration is max(0.03, 0.9*stepDuration) at 40, 120 and 220 BPM.
- Roll note on step: `onStep` with `stepIndex % 16 === n.start` starts one voice at the given `time`.
- Step wrap: `stepIndex` 16 plays step 0 again.
- Swing time: `onStep` uses the given `time` and nominal `stepDuration` for note durations.
- Voice cap: a 17th overlapping voice at the same time is dropped and the voices LCD never shows more than 16.
- Mixed layers: a ROLL note and a bass step on the same step start two voices, both counted towards the cap.
- Envelope with A + D longer than the note: for a 2000 ms decay at 120 BPM the bass gain automation is time-ordered and has no jump at note-off.
- Stop silences: after `onStop` no voice gain is above zero after the fade time, and the voice set is empty. Reverb and delay tails may ring out.
- Switch: `on.synth` lives in the project and not in `getState()`; switching off calls `onStop`, and switching on while running calls `onStart(time)`.
- No timers: `createInstrument` and `onStart` create no `setInterval` and no `setTimeout` for scheduling.
- State round trip: `setState(getState())` leaves `getState()` unchanged, and the state survives `JSON.stringify` and `JSON.parse`.
- Defaults: a new instrument has 21 demo notes, 16 bass rests, and the defaults of the controls table.
- Clamping: `setState` with out-of-range values (for example `pitch` 90, `degree` 12, `bass.octave` 9) clamps them to the ranges.
- Hidden or unmounted: `onStep` runs without `mount` and without a DOM, and the ring buffer holds no entry older than one second and nothing reads it.
- Delay time: at 120 BPM the "1/8 ." option gives 0.375 s, and a tempo change updates it at once through `transport.subscribe`.
- Reverb rebuild: ten fast changes of Reverb Decay trigger one impulse response rebuild, 150 ms after the last change.
- Output routing: all voice audio reaches `output` only, never `ctx.destination`.
- Knob keys and reset: arrow keys step by 1 (Shift x5); double-click resets knobs that have a reset value (Reverb Mix resets to 0) and does nothing on knobs with `reset: null` (Reverb Size).
- Randomise: RND of the BASS view gives velocities in 0.35..0.95 and degrees in 0..min(9, scaleLen + 2) - 1; RND of the ROLL view keeps all pitches in 48..83.
- Roll editing (browser test): double-click adds a note of length 1 and velocity 100; double-click on a note deletes it; arrow keys move the selected note within 48..83 and 0..16-len.

## Open questions

1. Both sources have a different level scale. SM peak is vel/127*0.26*waveLevel (about 0.14 for a full-velocity sawtooth). S3 peak is the velocity (0..1) with no compressor. The bass level factor is TBD; a value is needed so the bass sits well against the roll through the shared compressor.
2. Should the bass layer feed the reverb and delay of the roll (as specified here), or stay dry like in S3? The sources cannot settle it.
3. Should the compressor stay inside the instrument chain (as specified here, to keep the SM sound) or move to the master bus?
4. Do both layers play at the same time (as specified here) or does the view switch also act as a mute? S3 and SM are separate pages, so the sources do not say.
5. The fade length for `onStop` voice silencing is TBD (any value that avoids a click is fine; neither source has one).
6. The BASS view uses the S3 text "- rust -"; the spec shows "REST" for an English interface. Is that wording accepted?
