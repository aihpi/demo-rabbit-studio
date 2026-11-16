# Beat

## Sources

- BM: RABBIT-BeatMachine_x.html ("RABBIT Beat Machine", 16-step drum computer, 1733 lines). Report: report_RABBIT-BeatMachine_x.json. BM is the base of this module: five channels, pads, sends, reverb, delay, filters, VU.
- D4: CP5_TT_20251118_DrumSEQ4.html ("Tom's Beat Station", DrumSEQ4, 576 lines). Report: report_CP5_TT_20251118_DrumSEQ4.json. Every D4 function that BM lacks is added: fine per-step velocity, filter modes, randomise all, clear all, sound choice per channel, Space key, master level (now on the Console master bus).
- Original synthesis values (frequencies, envelopes, gains) are taken from the audioGraph entry of the BM report. Both originals were read as static reports only and were not run.

## Purpose

Beat is a five-channel, 16-step drum sequencer with synthesised drum voices or uploaded samples, per-channel mix controls, a global filter section, a send reverb, a tempo-synced send delay and a stereo VU meter. It follows the global transport of RABBIT Studio and keeps playing while another tab is shown. It combines the sound and look of BM with the editing functions of D4.

## Feature parity

Where a row says "header" the function moves to the global transport in the page header (start, stop, tempo, swing). Where it says "Console" the function moves to the Console (the instrument's channel strip or the master bus). Per-channel volume, pan, mute and solo inside Beat are internal to the instrument and stay in Beat. The Console strip acts on the whole Beat output; its mute and solo act on instruments and ignore Beat's per-channel solo. Rows that depend on the Console (master volume, post-strip meter) are ticked together with the Console at stage 6; every other row is ticked at stage 4.

| Feature in the source | Source | In RABBIT Studio |
|---|---|---|
| 5 fixed channels KICK, SNARE, HI-HAT, CYMBAL, PERC; synth voice unless a sample is loaded; names are editable (Enter or blur commits, empty text reverts) | BM | Kept. Defaults as BM. The name now also updates the aria labels live (see Rewrite and accessibility changes). D4 has only 3 channels, so BM wins. |
| 16 steps per channel in 4 groups of 4, ruler 1, 5, 9, 13, "PATTERN" label, all patterns start empty | BM | Kept. D4 also has 16 steps and bar-start marking on steps 1, 5, 9, 13, which the ruler covers. |
| Step button with 3 levels: off, full (1.0), half (0.5); click cycles off, full, half, off | BM | Kept as the click behaviour of the pad. The stored value is now a number from 0 to 1 (see next row), so full is 1.0 and half is 0.5. |
| Per-step velocity slider 0 to 1, step 0.01, default 0; velocity above 0 marks the step active, 0 is silent | D4 | Added. A velocity strip of 16 faders sits under each channel's pads. The pad fill height shows the velocity. Both edit the same value. BM's click cycle and D4's fine slider coexist because both write steps[i]. |
| Double-click on a cell flips velocity to 0 if above 0, else to 1 | D4 | Replaced by BM's single-click cycle, which is a superset and works on touch. D4's double-click is dropped because a single click already toggles. |
| Per-channel RND: each step on with probability 0.35, level 1 or 2 with 50/50; CLR zeroes all 16 steps | BM | Kept for the RND and CLR buttons per channel (probability 0.35, velocity 1.0 or 0.5). |
| Per-channel Random (density 0.5, velocity 0.4 to 1.0 per active step) and Clear | D4 | The Clear button is the same as BM's CLR. The D4 random distribution is used only by "randomise all" (next row). The per-channel RND keeps BM's distribution because BM is the base. |
| Global "Randomize alle kanalen" (density 0.45 to 0.65 per channel, random per channel) and "Clear alle steps" | D4 | Added as RND ALL and CLR ALL on the sequencer rack, applied to all 5 channels (D4 hard-coded 3). Distribution as D4. Samples and mix settings are not touched. Labels are English. |
| Per-channel MUTE and SOLO; isSilent = muted or (any solo and not this solo); silent channels at 50 % opacity; evaluated at scheduling time | BM | Kept. D4 has no mute or solo. Evaluated in onStep, so the late effect of up to the lookahead is the transport's lookahead, not a Beat setting. |
| Per-channel VOL knob 0 to 100 % (defaults 82, 78, 60, 55, 70) | BM | Kept as internal channel level. |
| Per-channel volume slider 0 to 1.5, step 0.01, default 1.0 | D4 | BM wins (knob 0 to 100 %, defaults as BM). D4's extra headroom up to 1.5 and its clipping risk are not carried over. |
| PAN knob minus 100 to plus 100, double-click resets to 0, readout C, nL, nR | BM | Kept. D4 has no pan. |
| Faders DECAY (per-channel ranges), PITCH minus 12 to plus 12 semitones (ratio 2 to the power st/12), RVB send 0 to 100 %, DLY send 0 to 100 % | BM | Kept. |
| Global decay ("Nootlengte") 0.03 to 1.5 s, default 0.25, scales every voice envelope | D4 | Not added as a separate control. BM's per-channel DECAY faders cover the same function with finer control, and one global value would conflict with them. Listed in Open questions. |
| Sample loading per channel (hidden file input audio/*, name truncated to 16 chars or "SYNTH", x clears, decode failure shows "LOAD ERROR") | BM | Kept. The file is uploaded to the backend and referenced by id in the state (see State). D4 has no samples. |
| Synth voices: kick (sine sweep plus square click), snare (high-passed noise plus two triangles), hi-hat and cymbal (six detuned squares through HPF and BPF), perc (triangle pitch drop) | BM | Kept as the sound of Beat. |
| Instrument synthesis: kick sine 140 to 45 Hz, snare bandpass noise plus 170 Hz triangle, hi-hat high-passed band-passed noise | D4 | BM voices win because BM is the base and has two more voices. The D4 voices are not offered as extra choices (Open questions). |
| Per-channel voice select Kick, Snare, Hihat (all three default to kick) | D4 | Added as a VOICE select per channel with the five BM voices (kick, snare, hihat, cymbal, perc). Defaults follow BM (channel 1 kick, 2 snare, 3 hihat, 4 cymbal, 5 perc), not D4's three kicks, because a fresh page should sound like a kit. |
| Transport START and STOP buttons; START disabled while playing, STOP while stopped; no pause, loop region or pattern length | BM | Moved to the header. Beat has no transport buttons. |
| Start/Stop toggle button and Space bar shortcut (Space toggles, preventDefault on the whole document) | D4 | The Space key is added to Beat: while the Beat tab is shown, Space toggles the global transport, except when focus is in a select, text input or button. The page header shows the button. See defects. |
| Tempo 40 to 300 BPM, default 120, knob plus and minus buttons (1, Shift 5) | BM | Moved to the header (40 to 300, BM range wins over D4's 40 to 220). Beat only reads the tempo. |
| Global tempo 40 to 220 BPM, default 120, step 1 | D4 | Moved to the header, see previous row. |
| Swing 0 to 70 %, default 0, even steps lengthened and odd shortened by stepDur times swing/100 times 0.5 | BM | Moved to the header. The transport passes the swung time to onStep. D4 has no swing. |
| Global filter HPF (Cut 0 to 100 default 0, Res 0 to 100 default 0) and LPF (Cut default 100, Res default 0); cut maps to 20 times (20000/20) to the power v/100 Hz; Q = lerp(0.7, 18, v/100) | BM | Kept as the dual filter. |
| Global filter on the summed mix: type lowpass, highpass or bandpass (default lowpass), frequency 200 to 12000 Hz default 4000, Q 0.1 to 24 default 1 | D4 | Added as a third filter stage "MODE" with select OFF, LOW PASS, HIGH PASS, BAND PASS, FREQ and Q knobs, after the BM dual filter. The default is OFF, not D4's lowpass 4000 Hz, so the BM sound stays and D4's quiet hi-hat default (see defects) does not occur. The ranges are D4's. |
| Master volume 0 to 1.2, step 0.01, default 0.9 | D4 | Moved to the master fader of the Console, whose range, taper and default are owned by docs/modules/shell.md and docs/modules/console.md; Beat states no master values. BM's fixed 0.9 master gain stays as an internal Beat bus gain of 0.9. Beat has no master control of its own. |
| BM has no master volume; masterGain fixed at 0.9 | BM | Same as previous row. |
| Reverb send: DECAY 0.2 to 4.0 s (default 1.6), SIZE 0 to 100 % (default 45), EQ (low-pass 300 Hz to 16 kHz, default 60), VOLUME minus 24 to plus 6 dB (default 0); IR is random noise with power-law decay, rebuilt when DECAY or SIZE changes | BM | Kept. Uses the shared fx/reverb.js; the IR is rebuilt debounced (see defects). D4 has no reverb. This is separate from the Console's own reverb send. |
| Delay send: TIME select with 11 divisions (1/32, 1/16 T, 1/16, 1/8 T, 1/16 dotted, 1/8, 1/8 dotted, 1/4, 1/4 dotted, 1/2, 1/1), default 1/8; FEEDBACK 0 to 95 % (default 32, real gain value/100 times 0.95); EQ (default 50); VOLUME minus 24 to plus 6 dB; ms readout recomputed on tempo change | BM | Kept. Uses the shared fx/delay.js, tempo-synced to the global tempo. |
| Stereo VU meter L/R, 10 segments each (1 to 6 green, 7 to 8 amber, 9 to 10 red), RMS of an AnalyserNode | BM | Kept with ui/vu.js, which registers with the shared loop in audio/meters.js on mount and has no loop of its own. The dB range is the one given in docs/modules/shell.md. The meter shows the Beat bus before the Console strip. The Console shows the post-strip level. |
| Stereo VU canvas 800 by 140, two horizontal RMS bars, level min(1, RMS times 1.9), placeholder "Wacht op audio…" until the graph exists | D4 | Replaced by BM's 10-segment meter (BM wins: look decided by the target application). The placeholder is dropped because the graph exists from createInstrument. |
| Sync box in the header: LED, status STOPPED / LOCKING... / RUNNING, countdown "STARTS IN x.xs", "LOCKED @ HH:MM:SS.mmm"; start waits for the next multiple of 4 s of the wall clock | BM | Removed by decision (wall-clock start and countdown). Running state is shown by the header transport. |
| Wall-clock start at the next second divisible by 4, status texts "Gestopt" / "Wachten op sync…" / "Spelend (gesynchroniseerd)", countdown text, header note | D4 | Removed by decision. |
| Playhead: ruler number and all pads of the current step get an outline; pads that fire flash for 130 ms | BM | Kept, driven from audio time through the ring buffer (see Transport behaviour). The ruler number turns accent red and the pads get the ink outline, as in BM. |
| Playhead highlight: class "now" (2 px green outline) on the current step column across all rows for max(30 ms, 0.8 step) | D4 | BM wins (outline on ruler and pads, 130 ms flash). The green colour is replaced by BM's ink outline. |
| Bar-start cells (steps 1, 5, 9, 13) get a lighter border | D4 | Covered by BM's groups of 4 and the ruler numbers 1, 5, 9, 13. |
| Decorative: power-on flicker, red hazard strip, four rivets, blind-shadow bars, footer legend "THE RHYTHM RACKET - CASE NO. 16 / 5 CH - 16 STEP - RVB + DLY" | BM | Hazard strip, rivets, blind-shadow bars and the plate look are kept as shared chassis styling. The flicker is kept only if prefers-reduced-motion is off. The footer legend is dropped: it is page chrome, not an instrument function (Open questions). |
| Not present in D4: mute/solo, swing, accent, pattern length change, more than 3 channels, sample loading, save/load, export, recording | D4 | Information entry. Mute, solo, swing (header), 5 channels and samples come from BM. Save and load come from the project backend. Accent, pattern length and export stay absent (no source has them). |
| Not present in BM: master volume, tap tempo, metronome, pattern save or load, export, MIDI, keyboard play | BM | Information entry. Master volume comes from the Console. Save and load come from the project backend. The metronome is its own instrument tab. The rest stays absent (no source has it that Beat needs). |

## Controls

| Control | Component | Range | Default | Source |
|---|---|---|---|---|
| Channel VOL (x5) | knob | 0 to 100 % | 82, 78, 60, 55, 70 | BM |
| Channel PAN (x5) | knob, double-click resets | minus 100 to plus 100 | 0 | BM |
| Channel DECAY (x5) | fader | kick 0.05 to 1.0 s, snare 0.05 to 0.6, hi-hat 0.02 to 0.3, cymbal 0.05 to 2.0, perc 0.05 to 0.8; step 0.01 | 0.35, 0.18, 0.06, 0.9, 0.25 | BM |
| Channel PITCH (x5) | fader | minus 12 to plus 12 semitones, step 1 | 0 | BM |
| Channel RVB send (x5) | fader | 0 to 100 % | 0, 15, 5, 30, 0 | BM |
| Channel DLY send (x5) | fader | 0 to 100 % | 0, 0, 0, 0, 20 | BM |
| Channel VOICE (x5) | select | kick, snare, hihat, cymbal, perc | kick, snare, hihat, cymbal, perc | D4 |
| Channel M and S (x5) | button (.btn--toggle) | off or on | off | BM |
| Channel RND and CLR (x5) | button (.btn--pill) | action | none | BM, D4 |
| Channel name (x5) | lcd (editable text) | text, empty reverts | KICK, SNARE, HI-HAT, CYMBAL, PERC | BM |
| Channel LOAD SMPL and x (x5) | button (.btn--pill) with file input audio/* | file; x clears | no sample ("SYNTH") | BM |
| Step pad (16 per channel) | pad | click cycles 0, 1.0, 0.5, 0 | 0 | BM |
| Step velocity (16 per channel) | fader (compact vertical size of ui/fader.js) | 0 to 1, step 0.01 | 0 | D4 |
| RND ALL, CLR ALL | button (.btn--pill) | action | none | D4 |
| HPF Cut | knob | 0 to 100 | 0 (20 Hz) | BM |
| HPF Res | knob | 0 to 100 | 0 | BM |
| LPF Cut | knob | 0 to 100 | 100 (20 kHz) | BM |
| LPF Res | knob | 0 to 100 | 0 | BM |
| Filter MODE | select | OFF, LOW PASS, HIGH PASS, BAND PASS | OFF | D4 (default decided here) |
| Filter FREQ | knob | 200 to 12000 Hz | 4000 | D4 |
| Filter Q | knob | 0.1 to 24, step 0.1 | 1.0 | D4 |
| Reverb DECAY | knob | 0.2 to 4.0 s, step 0.05 | 1.6 | BM |
| Reverb SIZE | knob | 0 to 100 % | 45 | BM |
| Reverb EQ | knob | 0 to 100 (low-pass 300 Hz to 16 kHz) | 60 (about 3.3 kHz) | BM |
| Reverb VOLUME | knob | minus 24 to plus 6 dB, step 0.5 | 0 | BM |
| Delay TIME | select | 11 divisions, 1/32 to 1/1 | 1/8 | BM |
| Delay FEEDBACK | knob | 0 to 95 % | 32 | BM |
| Delay EQ | knob | 0 to 100 (low-pass 300 Hz to 16 kHz) | 50 (about 2.2 kHz) | BM |
| Delay VOLUME | knob | minus 24 to plus 6 dB, step 0.5 | 0 | BM |
| Beat VU (L, R) | vu, 10 segments | meter | none | BM |
| Channel LEDs (x5) | led | lit for 130 ms on a hit | off | BM |
| Space key | keyboard | toggles global transport | none | D4 |

Tempo (40 to 300, default 120, with plus and minus buttons of 1 and Shift 5) and swing (0 to 70 %, default 0) are in the header and are not Beat controls. Master volume (D4 range 0 to 1.2, default 0.9) is the Console master fader and is not a Beat control; its values are owned by docs/modules/shell.md and docs/modules/console.md. Knob interaction (vertical drag, 150 px for the full range, wheel, arrow keys, Home and End) comes from ui/knob.js.

## State

One JSON object. Step values are numbers from 0 to 1. A non-null sampleId means the channel plays that sample instead of its synth voice.

```json
{
  "channels": [
    {
      "name": "KICK", "voice": "kick",
      "steps": [1, 0, 0, 0, 0.5, 0, 0, 0, 1, 0, 0, 0, 0.5, 0, 0.8, 0],
      "vol": 82, "pan": 0, "decay": 0.35, "pitch": 0,
      "rvb": 0, "dly": 0, "mute": false, "solo": false,
      "sampleId": null, "sampleName": null
    }
  ],
  "filter": { "hpCut": 0, "hpRes": 0, "lpCut": 100, "lpRes": 0, "mode": "off", "freq": 4000, "q": 1 },
  "reverb": { "decay": 1.6, "size": 45, "eq": 60, "vol": 0 },
  "delay": { "division": "1/8", "feedback": 32, "eq": 50, "vol": 0 }
}
```

The "channels" array always holds 5 entries, in the example only the first is shown. Tempo, swing and playing state are not in the state; they belong to the transport. The on/off switch of Beat is stored in the project as `on.beat`, not in this object. Which tab or view is shown is interface state and never enters the project. `schemaVersion` belongs to the project, not to this object. No audio nodes, DOM references or decoded buffers are stored.

## Audio graph

Per voice, created per hit and stopped at the envelope end (r = 2 to the power pitch/12):

- Kick: sine osc 160 times r Hz, exponential ramp to max(48 times r, 20) over 0.08 s, gain 1 to 0.0001 over max(decay, 0.05), stop at t plus decay plus 0.05; plus square click osc 800 times r Hz, gain 0.3 to 0.0001 in 0.02 s, stop at t plus 0.03.
- Snare: shared 2 s mono noise buffer (playbackRate r), highpass 900 times r, gain 0.85 to 0.0001 over max(0.85 times decay, 0.04); plus triangle oscs 180 times r and 330 times r, gain 0.4 to 0.0001 over max(0.5 times decay, 0.03).
- Hi-hat: 6 square oscs at 40 times r times [2, 3, 4.16, 5.43, 6.79, 8.21], mix gain 0.6, highpass 7000, bandpass 9000, env gain 1 to 0.0001 over max(decay, 0.02).
- Cymbal: as hi-hat with base 32, bandpass 7000, highpass 5000, mix gain 0.5.
- Perc: triangle osc 320 times r exponential to max(170 times r, 40) over 0.05 s, gain 0.85 to 0.0001 over max(decay, 0.05).
- Sample: BufferSource (playbackRate r), env gain 1 to 0.0001 over max(decay, 0.05), stopped at t plus decay plus 0.05.

The noise buffer comes from audio/noise.js. Chain per channel:

voice, then veloGain (the step velocity, 0 to 1; BM's 0.5 and 1.0 are two values of it), then ch.gain (vol/100), then ch.pan (StereoPanner), then beatBus. From ch.gain, a post-fader and pre-pan send goes to ch.rvbSend (rvb/100) then reverbBus, and to ch.dlySend (dly/100) then delayBus.

Beat bus and output:

- beatBus (gain 0.9) then HPF (biquad highpass) then LPF (biquad lowpass) then modeFilter (biquad; type allpass when mode is OFF) then output (the strip input that the engine hands to Beat; Beat never connects to ctx.destination).
- The modeFilter then also feeds ChannelSplitter(2), AnalyserL and AnalyserR (fftSize 256) for the Beat VU. Analysers are not connected onward.
- Reverb: reverbBus, ConvolverNode (fx/reverb.js, noise IR), lowpass EQ, out gain (dB), then beatBus.
- Delay: delayBus, DelayNode (fx/delay.js, time 60/bpm times multiplier, max 8 s), lowpass EQ, out gain (dB), then beatBus. The delay filter also feeds a feedback gain (value/100 times 0.95) back into the DelayNode.
- Reverb and delay returns enter beatBus, so they pass through the filters, as in BM.

Beat maps its knobs to the shared modules like this: createReverb(ctx, { decay, preDelay, power, toneHz, mix }) with DECAY as decay, SIZE as preDelay and power, EQ as toneHz and VOLUME as mix; createDelay(ctx, transport, { division, feedback, toneHz, mix }) with TIME as division, FEEDBACK as feedback, EQ as toneHz and VOLUME as mix. Beat passes its own values, so it keeps the reverb sound of BM.

All nodes are created in createInstrument, not in mount. Nothing in the graph depends on mounted DOM.

## Transport behaviour

Start: onStart(time) resets the ring buffer of visual cues and clears pending voices. No wall-clock wait and no countdown; the first step arrives from the transport at the transport start time. Step 0 is on time (no 25 ms lateness of D4).

Each step: onStep(stepIndex, time, stepDuration) does audio only and uses i = stepIndex modulo 16. For each channel that is not silent (mute and solo rule) and has steps[i] above 0, it starts the voice (or sample) at exactly `time` with veloGain = steps[i]. It writes { stepIndex, time } into a ring buffer (max 64 entries, oldest dropped). Swing is already in `time` and `stepDuration`. A mounted interface reads the buffer in the shared animation loop of audio/meters.js, drops entries older than one second, handles entries whose time is not after ctx.currentTime, sets the playhead outline and a 130 ms flash on pads, and lights the channel LED. An unmounted Beat, or Beat in an offline render, has no reader and does no interface work.

Stop: onStop() cancels every voice scheduled but not yet started (so no note starts after Stop), ends every sounding voice with a short fade (about 20 ms, gain ramp to zero, then stop), and clears the ring buffer and the playhead. Reverb and delay tails may ring out. This is plan.md rule 8.

Tempo change: the transport reads the new tempo on the next step. fx/delay.js retimes the delay itself through transport.subscribe, also while the transport is stopped; Beat does no retiming. Beat reads transport.bpm only to update the ms readout of the delay. Swing changes also apply from the next step (transport).

While the tab is hidden: audio is unaffected, because Beat schedules audio only in onStep and holds no DOM dependency. On unmount() the interface unregisters from the shared animation loop, removes its knob listeners and discards the ring buffer. On mount() the playhead resumes from the current step. Hidden-tab timing is the transport's job: its 25 ms tick runs in a Web Worker with a lookahead of 0.2 s (docs/plan.md, section Transport).

Switch: the on/off switch of Beat is stored in the project as `on.beat`, not in the Beat state. Switching off calls onStop; tails of the last notes may ring out. Switching on while the transport runs calls onStart(time), and Beat plays from the next onStep with the current stepIndex. The Console has no switch.

## Defects fixed

Each defect in this section broke behaviour and gets one line in CHANGELOG.md with the file and line it came from (plan.md rule 10).

- STOP does not stop already scheduled notes, and onStepPlay can re-add the playhead and flip the status back to RUNNING after STOP (BM report, scheduling and weaknesses, lines 1700-1713, 1657-1663). D4 also leaves notes sounding after Stop (D4 scheduling, line 449). Fixed by onStop() as above, which also fades sounding voices.
- Duplicate scheduler: Start, Stop, Start within the wait leaves two timeouts and two intervals, every note sounds twice and audio cannot be stopped (D4 weaknesses, lines 428, 429, 432, 447-453; Space repeat at line 535). Fixed by removal of the wall-clock wait and the single transport scheduler; Beat has no timer of its own. The Space handler ignores repeated keydown events.
- Stale countdown and sync status after Stop during the wait (D4, lines 425, 429, 434; BM, lines 1662, 1666-1670). Removed with the feature.
- Step 0 plays late by about 25 ms (D4, lines 431, 432). Fixed: the transport supplies the time for every step.
- No catch-up guard: after a main-thread stall steps with past times play as a burst (BM, lines 1082-1087). Fixed once in the transport, which skips a step more than 5 ms late. Beat has no guard of its own.
- Anchor sampled while the AudioContext is suspended, first start offset (BM, lines 1673-1677). Gone with the wall-clock mapping.
- Reverb IR rebuilt synchronously on every knob input event, with jank and clicks (BM, lines 848-852, 1509, 1513). Fixed: fx/reverb.js rebuilds debounced (about 150 ms after the last change).
- Channel volume slider ignored before the first Start (D4, lines 244, 370). Fixed: the graph exists from createInstrument and is built from the state.
- Highlight runs at scheduling time and leads the sound by up to 120 ms (D4, line 397; BM report: visuals lead audio, lines 1716-1730). Fixed: the playhead is drained at audio time.
- Two rAF loops never cancelled, and the visual queue grows unbounded in a hidden tab (BM, lines 783-793, 1716-1730, 1073). Fixed: Beat has no loop of its own, it registers with the shared loop only while mounted; the ring buffer is capped at 64.
- Re-picking the same file after a load fires no change event (BM, lines 1421-1434, 1436). Fixed: the input value is reset after every selection.
- Decode failure shows only a label and the control can throw before a file is loaded (BM, line 1430). Fixed: the sample controls work with no sample; a failed upload or decode shows "LOAD ERROR" and leaves the synth voice active.
- Space handler calls preventDefault on every Space press, also in selects and on the focused button, risking double toggling (D4, line 535). Fixed as described under Feature parity.
- Filter and master parameters set with .value, giving zipper noise (D4, lines 520-531). Fixed: filter parameters change with setTargetAtTime (about 10 ms).
- Filter default lowpass 4000 Hz removes most of the hi-hat energy (D4, lines 161, 336-337). Fixed: the MODE default is OFF.
- Right-click starts a knob drag and the wheel handler always blocks page scroll (BM, lines 1260, 1283). Fixed in the shared ui/knob.js (pointer capture, button check).

## Rewrite and accessibility changes

These clean-ups come with the rewrite. They did not break behaviour, so they are not in CHANGELOG.md.

- Stale aria-labels for renamed channels (BM, lines 1314, 1322, 1332, 1342). Labels read the current name.
- Hard-coded channel indices [0, 1, 2] and DOM-order coupling (D4, lines 542, 551, 557, 560, 566). All channel loops use the length of state.channels.
- Knobs have no aria-valuetext (BM, accessibility). Added in ui/knob.js.
- Decay, filter and per-channel controls have no label, and the playhead and active state are shown by colour only (D4, lines 143, 148, 152, 232-245, 263-264). Every control has a visible label or an aria-label, and the playhead outline and the pad fill carry the state besides colour.
- Dutch labels and status texts hard-coded in the script (D4, lines 131, 174, 433, 442, 451). The interface language is English.

## Kept on purpose

- The HPF Res and LPF Res mapping up to Q 18 (BM, line 834), with its large resonance peak. There is no limiter and no compressor, as in BM.
- Post-fader, pre-pan mono sends (BM, lines 937-952).
- Reverb and delay returns pass through the HPF and LPF (BM, lines 908-935).
- The random noise IR (non-deterministic, power-law decay), as in BM.
- A loaded sample is cut at decay plus 0.05 s, so the DECAY fader also trims samples (BM, line 1031).
- Even steps lengthened and odd steps shortened by swing, with the pair summing to 2 steps (BM, lines 1077-1078). Done by the transport.
- The shared noise buffer: every snare hit has the same noise waveform (BM, 2 s buffer; D4, 1 s buffer).
- Hi-hat and cymbal as six detuned squares through HPF and BPF.
- VU colours: segments 1 to 6 green, 7 to 8 amber, 9 to 10 red. The meter is an RMS approximation, flickers a little because the analyser smoothing has no effect on time-domain data, and is not dB-calibrated (BM, lines 769-795).
- Channel names change labels only; they do not change the voice.
- Per-hit node creation (veloGain plus up to 8 oscillators for hats); nodes end by their stop time.

## Layout

```
+--[ hazard strip ]---------------------------------------------------------+
| BEAT                          ( RNDALL ) ( CLRALL )                       |
+-- MASTER ---------------------------+-- FILTER ----------------------------+
| BEAT  | VU L ||||||||||  VU R |||||| | HPF (Cut)(Res)  LPF (Cut)(Res)       |
| BUS   |                              | MODE [OFF v] (Freq) (Q)              |
+-- REVERB ---------------------------+-- DELAY -----------------------------+
| (Decay)(Size)(EQ)(Vol)              | TIME [1/8 v] (Feedback)(EQ)(Vol) 250ms|
+-- SEQUENCER ----------------------------------------------------------------+
|  PATTERN   1   .  .  .   5   .  .  .   9   .  .  .  13   .  .  .          |
+-----------------------------------------------------------------------------+
| (o) KICK    [pad][pad][pad][pad] [pad]... x16 in 4 groups of 4   [LOAD SMPL] |
| M S RND CLR [vel][vel][vel][vel] [vel]... x16                    SYNTH  [x]  |
| (VOL)(PAN)  DECAY | PITCH | RVB | DLY          VOICE [kick v]               |
+-----------------------------------------------------------------------------+
| (o) SNARE   ... same plate, 5 plates in total (darkening by row) ...        |
+-----------------------------------------------------------------------------+
Header (outside this module): [START] [STOP] BPM [120] SWING [0]
Console tab: channel strip for Beat (gain, pan, EQ, reverb send, mute, solo) and master bus
Accent red as in BM (token in styles/tokens.css): hazard strip, current ruler number, active M, knob pointers, top VU segments. Amber (token) for solo and mid VU segments. The pad playhead outline is ink.
```

The top two racks sit in two columns. The channel plates use the RABBIT chassis (page, accent red and amber for solo come from styles/tokens.css, Michroma and Barlow). Channel colours are as in BM. The 1180 px fixed width of BM is replaced by a fluid layout down to 640 px, the smallest supported width; there are no rules or tests for narrower screens. No colour value appears in the code of Beat (plan.md rule 6); the Beat styles live in styles/components.css and styles/chassis.css.

## Tests

All tests are pytest files in 02_backend/tests/, run with `uv run pytest`. Browser tests use Playwright for Python. The checks below on createInstrument, onStep and the state run inside the page against the real module (module-level checks, offline context where audio is meant). Live checks use `?test=1` with `window.__rabbit` and its onStep log.

- Defaults: createInstrument().getState() equals the default state of this spec (5 channels, defaults, filter OFF).
- State round trip: setState(getState()) leaves getState() identical, and an unknown or missing field falls back to the default.
- State is JSON: JSON.parse(JSON.stringify(getState())) equals getState(); no node or DOM object inside.
- Step cycle: a click on a pad goes 0, 1.0, 0.5, 0; a velocity slider value of 0.37 is stored as 0.37.
- Velocity scaling: a step of 0.5 starts the voice with veloGain 0.5; a step of 0 starts nothing.
- onStep with an empty pattern starts no voice; with steps[3] above 0 on channel 2, onStep(3, t, d) starts exactly one voice at t (offline context).
- onStep uses stepIndex modulo 16 (index 19 plays step 3).
- Mute and solo: a muted channel does not sound; with one solo only that channel sounds; solo plus mute on the same channel is silent.
- Voice select: each of the 5 voices starts the node set listed under Audio graph; a loaded sample replaces the voice; sampleId null returns to the voice.
- Pitch: ratio 2 to the power st/12 is applied to the kick start frequency (160 times r) for pitch 12 and minus 12.
- Randomise: RND on one channel gives about 35 % active steps over many runs with velocities only 1.0 or 0.5; RND ALL gives 45 to 65 % per channel with velocities 0.4 to 1.0; CLR ALL zeroes all 5 channels.
- Filter mapping: cut 0 gives 20 Hz, cut 100 gives 20000 Hz; Res 100 gives Q 18; mode OFF sets type allpass; modes set lowpass, highpass, bandpass with the D4 FREQ and Q.
- Delay time: division 1/8 at 120 BPM gives 0.25 s; at 100 BPM gives 0.3 s; all 11 multipliers match the BM table; feedback 100 never gives gain above 0.95.
- Reverb rebuild: 10 DECAY changes in 50 ms give at most 1 IR rebuild (debounced).
- onStop: voices scheduled for the future are cancelled; no voice starts after onStop; the ring buffer is empty; no voice has a gain above zero after the fade.
- Switch: switching off calls onStop; switching on while the transport runs calls onStart(time) and plays from the current stepIndex; the switch is stored as on.beat and is not in getState().
- Hidden or unmounted: with no mount, onStep still produces audio; mount then unmount leaves Beat unregistered from the shared loop in audio/meters.js.
- Ring buffer: after 100 onStep calls with no reader it holds at most 64 entries.
- Same step, same time: with ?test=1, Beat and the Metronome log the same stepIndex at the same audio time.
- Sample load: the same file picked twice triggers two uploads; a decode failure shows LOAD ERROR and keeps the synth voice.
- Space key: toggles the transport when focus is on the body; does not toggle when focus is in a select; a repeated keydown toggles once.
- Rename: an empty name reverts; a new name updates the aria labels at once.
- Playwright for Python: load the page with ?test=1, switch Beat on, click pads, press Start; check that the playhead outline moves, switching to another tab does not stop the audio (AudioContext time advances and the onStep log keeps growing), and Stop silences it.

## Open questions

1. Global decay (D4, 0.03 to 1.5 s, default 0.25): should a global decay trim be added next to the per-channel DECAY faders? This spec says no; no numbers exist for a combined behaviour.
2. D4 voice variants (bandpass snare with 170 Hz triangle, noise hi-hat): should they be selectable as extra voices? This spec offers only the five BM voices.
3. Filter MODE default: this spec uses OFF. D4's default (lowpass 4000 Hz) makes the hi-hat very quiet. Is OFF acceptable?
4. Should the per-channel RND also use D4's distribution (density 0.5, velocity 0.4 to 1.0), or stay with BM's 0.35 and two levels?
5. Samples: should Beat decode the file once per setState, or keep the decoded buffer until the sampleId changes? The sources do not define it (TBD). The routes and limits are in docs/plan.md, section Backend.
6. The loaded sample is cut at decay plus 0.05 s (BM line 1031). This is kept; should long samples play to the end instead?
7. Layout: BM uses a fixed 1180 px stage and 156 px fader slots; the plan sets 640 px as the smallest width. The velocity strip under the pads is a layout decision of this spec, not from either source.
8. Footer legend "THE RHYTHM RACKET - CASE NO. 16": dropped here as page chrome; confirm.
9. The Space shortcut: Beat adds it while the tab is shown. Should the app shell own Space for all tabs instead?
10. Does the Console reverb send duplicate Beat's own reverb and delay sends? Both are kept; Beat's sends are internal to the instrument.
11. Output latency compensation for the playhead (BM, visuals lead audio): not fixed here because neither source gives a number (TBD).
