<!-- Written by automated readers from the source code of the files in 03_originals/. Nothing was run in a browser; treat behaviour claims as read from code, not measured. -->

# Inventory of eight single-file instruments (rebuild brief)

Source: eight static analyses of single-file HTML pages. Only one report states that it ran nothing (DrumSEQ4: "Nothing was run; this is a static read"). SynthMachine's report says "Read in full; not run", and several reports mark claims "unverified". Treat behaviour claims as read from code, not measured. Line references use the form "l.N" and refer to the file named in each case.

Column keys used in the tables: **D4** DrumSEQ4, **S3** SynthSEQ3, **SL** Slicer5, **TH** theremin01, **MT** Metronoom5, **BM** RABBIT-BeatMachine_x, **SM** RABBIT-SynthMachine1_Full, **MX** RBBT-MixConsole1_Full.

## 1. Instruments

| Key | File | Instrument (page title) | Generation | Purpose |
|---|---|---|---|---|
| D4 | CP5_TT_20251118_DrumSEQ4.html | Tom's Beat Station | early "CP5" | 3-channel x 16-step drum sequencer (kick, snare, hi-hat synthesised), one global filter, VU canvas, start delayed to the next 4 s wall-clock boundary. |
| S3 | CP5_TT_20251118_SynthSEQ3.html | Tom's Bass Station | early "CP5" | 16-step monophonic bass sequencer (scale degree + velocity per step), one shared lowpass, ADSR, 4 s sync start. |
| SL | CP5_TT_20251118_Slicer5_update20260524.html | Web Sampler / Random Slice Sampler | early "CP5" | Loads one audio file, cuts it into equal slices, plays random slices on a 16th-note grid with filter, ADSR and reverb; 4 s sync start. |
| TH | CP5_FD_20251104_theremin01.html | Webcam Theremin | early "CP5" | Monophonic theremin: webcam brightness centroid Y sets pitch, X sets volume. Not a sequencer. |
| MT | CP5_TT_20251119_Metronoom5.html | Metronoom & Slot Machine | early "CP5" | One-voice metronome plus an unrelated random-number slot machine. Not a step sequencer. |
| BM | RABBIT-BeatMachine_x.html | RABBIT Beat Machine ("full version 1.0") | later "RABBIT" | 5-channel, 16-step drum computer with samples, swing, filters, reverb and delay sends, stereo VU; 4 s wall-clock start. |
| SM | RABBIT-SynthMachine1_Full.html | RABBIT Synth Machine | later "RABBIT" | 16-step, 36-pitch piano-roll polysynth with HPF/LPF, compressor, reverb, delay; 4 s wall-clock start. |
| MX | RBBT-MixConsole1_Full.html | RABBIT Studio Console ("Class-A Magic Mix") | later "RABBIT" (file prefix "RBBT") | Multitrack mixing console: up to 32 loaded audio files, EQ, pan, fader, reverb send, looping timeline, WAV mixdown. No sequencer. |

Notes on the table: generation follows the file prefix (CP5 vs RABBIT/RBBT) and the visual language in section 5. The theremin uses the prefix "CP5_FD", the others "CP5_TT"; the reports do not explain the "FD"/"TT" difference. Sizes: D4 576 lines, S3 585, SL 513, TH 373, MT 159, BM 1733 (91 KB), SM 1240 (81 KB), MX 1314 (about 71 KB).

## 2. Feature matrix

"x" = present. "(p)" = present with a limit given in the notes under the table. Blank = absent.

| Feature | D4 | S3 | SL | TH | MT | BM | SM | MX |
|---|---|---|---|---|---|---|---|---|
| **Transport** | | | | | | | | |
| Start / Stop | x | x | x | x (camera) | x | x | x | x |
| Tempo (BPM) | x | x | x | | x (p) | x | x | |
| Live tempo change | x | x | x | | | x | x | |
| Swing 0-70 % | | | | | | x | x | |
| Wall-clock start on 4 s boundary | x | x | x | | | x | x | |
| Sync status text / countdown | x | x | x | | | x (LED + countdown) | x (LED + countdown) | |
| Playhead / step highlight | x | | | | x (beat dot) | x | x | x (timeline) |
| Loop region, seek, rewind | | | | | | | | x |
| **Sequencing** | | | | | | | | |
| 16-step grid | x | x | | | | x | x | |
| Multiple channels (3 / 5) | x (3) | | | | | x (5) | | x (6-32 tracks, not steps) |
| Per-step velocity | x (slider) | x (slider) | | | | x (3 levels) | x (velocity lane) | |
| Per-step pitch | | x (scale degree) | | | | | x (piano roll) | |
| Piano roll (36 pitches, note length) | | | | | | | x | |
| Random slice triggering with density | | | x | | | | | |
| Randomise / Clear pattern | x (per channel + all) | x | | | | x (per channel) | x | |
| Scales (4) and octave select | | x | | | x (octave only) | | x (octave shift only, no scales) | |
| Demo pattern at load | | | | | | | x | |
| Slot machine | | | | | x | | | |
| **Sound sources** | | | | | | | | |
| Synthesised drums | x (3 voices) | | | | | x (5 voices) | | |
| Oscillator synth voice | | x | | x | x (click) | | x | |
| Waveform select | | x (3) | | | x (3) | | x (4) | |
| Detune / polyphony cap | | | | | | | x (both) | |
| Sample loading | | | x (one file) | | | x (per channel) | | x (per track) |
| Webcam input | | | | x | | | | |
| Per-channel pitch (semitones) | | | | | | x | | |
| **Shaping** | | | | | | | | |
| ADSR amplitude envelope | | x | x | | | | x | |
| Decay / note-length control | x | | | | x | x (per channel) | | |
| Filter (shared, with resonance) | x (LP/HP/BP) | x (LP) | x (LP/HP/BP/notch) | | x (LP, per click, no Q) | x (HPF + LPF) | x (HPF + LPF) | |
| Per-track EQ (lo-cut, 3-band) | | | | | | | | x |
| Slice length and jitter | | | x | | | | | |
| Smoothing / threshold (vision) | | | | x | | | | |
| **Effects** | | | | | | | | |
| Reverb (generated noise IR) | | | x | | | x (send) | x | x (send/return) |
| Delay (tempo-synced) | | | | | | x | x | |
| Compressor | | | | | | | x | |
| **Mixing** | | | | | | | | |
| Master volume | x | x | x | x (max volume) | x | (p) fixed 0.9 | x (dB) | x |
| Per-channel volume | x | | | | | x | | x |
| Pan | | | | | | x | | x |
| Mute / solo | | | | | | x | | x |
| Per-channel reverb / delay send | | | | | | x (both) | | x (reverb only) |
| **Metering** | | | | | | | | |
| Level meter | x (canvas, L/R RMS) | x (DOM bar, mono RMS) | x (DOM bar, dB, "CLIP") | | | x (10-segment L/R) | x (10-segment L/R) | x (canvas peak per strip + master L/R) |
| Waveform / slice display | | | x | | | | | |
| Envelope scope | | | | | | | x | |
| Webcam HUD (crosshair, ring) | | | | x | | | | |
| **File IO** | | | | | | | | |
| Audio file import | | | x | | | x | | x |
| WAV export (offline mixdown) | | | | | | | | x |
| Camera access | | | | x | | | | |
| Save / load of patterns or projects | | | | | | | | |
| **Other** | | | | | | | | |
| Rotary knobs (custom) | | | | | | x | x | x |
| Keyboard interaction on controls | x (Space) | | | | | x (knobs) | x (knobs, roll) | |
| Editable channel / track names | | | | | | x | | x |
| Responsive rules | | | | x (860 px) | | | x (1000, 640 px) | |
| prefers-reduced-motion | | | | | | x | x | |

Notes on "(p)": MT reads tempo only at Start (l.130); its live change is ignored. BM has no master volume control, only a fixed `masterGain` of 0.9 (l.883). MT filter is a fixed lowpass with cutoff slider and no resonance control. Persistence row: no file saves state, so the row is empty by design; it is a gap shared by all eight.

## 3. Overlap and gaps

### Features that exist several times in different implementations

- **Lookahead scheduler (25 ms timer, 0.12 s ahead)**: D4, S3, BM, SM. SL uses the same pattern with 0.25 s lookahead (l.211). MT and TH have none; MX has none for steps and uses rAF-driven loop restarts.
- **Wall-clock 4 s start**: three different implementations. D4 and S3 compute a delay with `setSeconds` and start through `setTimeout`. SL converts the delay into an absolute audio time (l.419-421). BM and SM anchor `ctx.currentTime` to `Date.now()` and compute `Math.ceil((anchorWall+150)/4000)*4000`.
- **Step sequencer**: four variants (D4 slider cells, S3 select + vertical slider, BM three-level pads, SM canvas piano roll).
- **Filter**: shared single filter in D4, S3, SL; HPF + LPF pair in BM, SM; per-click filter in MT; per-track EQ in MX.
- **Reverb**: four separate generated-noise impulse responses (SL, BM, SM, MX), each rebuilt on knob input (MX debounces 120 ms, l.988-990).
- **Delay**: BM and SM, with the same idea (tempo-synced, feedback max 0.95 times value) but different division lists (BM 11, SM 10).
- **VU meter**: six implementations: canvas RMS (D4), DOM bar RMS (S3), DOM bar with dB and CLIP (SL), 10-segment DOM (BM, SM), canvas peak (MX).
- **ADSR**: S3, SL, SM, each with different ranges and ramp types.
- **Knob and fader control code**: BM and SM share the `wireKnob` interaction (150 px drag, wheel, arrows, Home/End); MX has its own `makeKnob` (140 px drag, no keyboard).
- **Sample decode**: SL, BM, MX each decode with `decodeAudioData` separately.
- **Randomise**: D4, S3, BM, SM, each with its own density rule.
- **Mute/solo, pan, sends**: BM and MX, with different send tapping (BM post-fader pre-pan; MX post-mute).
- **Swing**: BM and SM, same formula (`stepDur*swing/100*0.5`).

### Features that exist only once

- Piano roll, polyphony cap, detune, compressor, envelope scope, octave shift without scales: SM.
- Scales (four) and degree-per-step: S3.
- Random slice sampler, waveform/slice display, density and jitter: SL.
- Webcam vision control: TH.
- Slot machine: MT.
- Per-track EQ, lo-cut, 32 tracks, loop timeline, WAV export, peak-hold readouts: MX.
- Per-channel pitch, per-channel pan and sends (together with mute/solo), editable channel names with samples per channel: BM (pan and mute/solo also in MX).
- Three-level step pads: BM.

### Gaps across all eight

No project save/load, no undo, no shared state between pages, no MIDI, no recording of live output, no shared transport. MT report: no features beyond those listed (no accent, presets).

## 4. Clock synchronisation

Common to all five syncing files: alignment is **at start only**, against the system clock (`Date`), never re-corrected afterwards. The reports state this for each file. TH, MT and MX have no clock logic at all (TH: "A search of the file for Date, performance.now ... finds nothing"; MT: "no Date.now, performance.now, new Date, getSeconds"; MX: "The only Date use is the download file name (l.1245)").

### D4 (DrumSEQ4), `startSchedulerAligned()` l.412-436

Quoted lines: `let add = (4 - (sec % 4)) % 4;` (l.417), `target.setSeconds(sec + (add===0 && ms===0 ? 0 : add || 4), 0);` (l.420), `const delayMs = Math.max(0, target - now);` (l.421), and one `setTimeout` at l.428 that sets `state.currentStep = 0; state.nextNoteTime = ctx.currentTime;` and starts `setInterval(schedulerTick, 25)` (l.431-432).

- Precision: the grid is anchored when the timeout fires, so setTimeout jitter (a few ms, more under load) enters directly. `nextNoteTime = ctx.currentTime` (l.431) is already past when the first interval tick runs 25 ms later, so step 0 sounds about 25 ms late. If `sec%4==0` and `ms>0` the wait is a full 4 s (report: "add||4").
- Drift: no outputLatency/baseLatency compensation; different BPM diverges at once; tempo change after start is a permanent shift; with 16 steps = 240/BPM s per loop, pages started on different 4 s boundaries align only for BPM 60, 120, 180 (and 240/BPM = 4/k); at 100 BPM (2.4 s loop) a page started 4 s later is 1.6 s out of phase; hidden tabs clamp timers (about 1 s).
- Defect that affects sync: Stop does not clear the pending timeout (l.428; stop l.447-453), so Start, Stop, Start in the wait window gives two schedulers.

### S3 (SynthSEQ3), l.530-550

Quoted: `const mod = now.getSeconds() % 4;` (l.532), `let addSeconds = (4 - mod) % 4; // 0 if already multiple of 4` (l.533-534), `target.setSeconds(now.getSeconds() + addSeconds); target.setMilliseconds(0);` (l.535-536), `const waitMs = Math.max(0, target.getTime() - now.getTime());` (l.536), then `setTimeout(() => { isRunning = true; currentStep = 0; nextNoteTime = audioCtx.currentTime + 0.01; scheduler(); ...}, waitMs);` (l.544-550).

- Precision: when `mod == 0`, the target is now with milliseconds zeroed (in the past), so the page starts at once at an arbitrary 0-999 ms offset from the boundary while the status still says "synced" (report, l.532-536, 549). Otherwise: timer lateness plus 10 ms, no latency compensation.
- Drift: no resync; step grid aligned across 4 s windows only if 4*BPM/15 is an integer (BPM multiple of 15), bar grid only for multiples of 60; a new page cannot join mid-pattern (every start sets `currentStep = 0`). The start `setTimeout` is not stored, so Stop during the wait does nothing.

### SL (Slicer5), l.411-421

Quoted: `const toNext = (4 - (secs % 4)) % 4;` (l.415), `let delayMs = toNext*1000 - ms; if (delayMs < 0) delayMs += 4000;` (l.416-417), `const startAt = audioCtx.currentTime + delayMs/1000;` (l.419), `setTimeout(()=> startSequencer(startAt), Math.max(0, delayMs));` (l.421).

- Precision: best of the three timeout-based files: the absolute audio time `startAt` is passed in, so timeout lateness does not move the grid (report). The offset is measured once at click time after `await audioCtx.resume()` (l.409). If the timeout fires late, the while loop schedules past ticks immediately as a burst.
- Drift: no output latency compensation; tempo per tab; tick = 15/BPM s, 4 s is an integer number of ticks only for BPM multiples of 15 (120 gives 32, 100 gives 26.67). Slices and density are random per tab (`Math.random`, l.443-444), so tabs share only the grid, not the content. Stop during the countdown does not cancel the start timeout (no handle).

### BM (BeatMachine), START handler l.1672-1698

Quoted: `anchorAudio = ctx.currentTime;` (l.1676), `anchorWall = Date.now();` (l.1677), `var bufferMs = 150;` (l.1678), `var gridMs = GRID_SEC*1000;` (l.1679), `var boundaryWall = Math.ceil((anchorWall+bufferMs)/gridMs)*gridMs;` (l.1680), `nextStepTime = audioTimeForWallMs(boundaryWall);` (l.1681), mapping `function audioTimeForWallMs(wallMs){ return anchorAudio + (wallMs - anchorWall)/1000; }` (l.1064). `GRID_SEC` is declared at l.728. The first boundary is 0.15-4.15 s away.

- Precision: the step is scheduled on the audio clock for the boundary, so setTimeout jitter is not in the grid. Residual error: the two clocks are read non-atomically (l.1676-1677); if the AudioContext was created in the same click and is still suspended, `currentTime` is frozen and the mapping can be off by the start-up delay (report items 4-5, l.1673-1677); output latency is ignored. The 'LOCKED @' time is printed from the first step (l.1662).
- Drift: no further correction ("After the first step there is NO further wall-clock correction", l.1079); tempo is local (l.1582-1588); BPM must be a multiple of 60 for pages started at different boundaries (period 240/BPM s); audio hardware clocks drift a few ms per minute on different devices (stated as reasoning, not measured); swing does not drift (even/odd pair sums to 2*stepDur).

### SM (SynthMachine), l.1201-1204

Quoted: `GRID_SEC = 4` (l.469), `const audioTimeForWallMs = ms => anchorAudio + (ms - anchorWall)/1000;` (l.1168), `anchorAudio = ctx.currentTime; anchorWall = Date.now();`, `const gridMs = GRID_SEC*1000;`, `const boundary = Math.ceil((anchorWall + 150)/gridMs)*gridMs;`, `nextStepTime = audioTimeForWallMs(boundary);`. UI text l.332: "Start locks to the next multiple of 4s on the system clock, then the pattern runs freely at your tempo".

- Precision: same mechanism as BM; `ctx.resume()` is not awaited (l.598 vs l.1201), so the suspended-context race exists here too. Date.now() has 1 ms resolution; `ctx.currentTime` advances in render quanta of about 2.7 ms, giving a few ms start offset (report).
- Drift: tempo 120 vs 121 BPM gives 2.000 s vs 1.983 s loops (report example); a mid-run tempo change accumulates a permanent offset and restoring the tempo does not realign (l.1178-1180); the "LOCKED @" text prints the planned boundary, not a measured time (l.1192-1195); the UI claim of staying in phase (l.332) holds only at identical tempo and clocks.

### Cross-file summary

| File | Anchor | Where jitter enters | Realigns? |
|---|---|---|---|
| D4 | setTimeout, then `ctx.currentTime` | timer lateness, +25 ms first tick | no |
| S3 | setTimeout, then `currentTime + 0.01` | timer lateness; immediate start when `mod==0` | no |
| SL | absolute audio time computed at click | late timeout makes a burst only | no |
| BM, SM | wall-to-audio anchor, boundary 150 ms+ ahead | suspended-context freeze, two non-atomic reads | no |
| TH, MT, MX | none | n/a | n/a |

Where two pages would drift, in all five syncing files: different or changed tempo; BPM not dividing the 4 s grid when pages start at different boundaries; hidden tabs (timer throttled to about 1 s against 0.12 s lookahead, 0.25 s in SL); output latency differences; audio-hardware versus system-clock rate differences (tens of ppm, reasoning only); system clocks not NTP-synced across machines.

## 5. Design system

### RABBIT generation (BM, SM, MX)

"Noir hardware chassis": dark metal plates, red accent, amber secondary, knobs and rivets.

Shared colours with roles (hex as stated in the reports):

| Role | Hex | Where |
|---|---|---|
| Page base | #131415 | BM, SM (`--bg`); MX calls it `--wood-dark` |
| Floor stripes (2 px) | #1c1d1f / #101112 | BM, SM, MX body |
| Chassis gradient | #202226 to #101112 | BM, SM `.unit` |
| Recessed wells | #1a1c1e | BM, SM (`--chassis-recess`); MX `--metal-dark` |
| Raised plates | #3a3d42 | BM, SM (`--chassis-raised`) |
| Border / stroke | #585c62; bright #82868d | BM, SM |
| Text primary | #f0f0ee | BM, SM (`--ink`); MX `--cream` |
| Text secondary | #b9bec5 | BM, SM (`--ink-dim`); MX `--cream-dim` |
| Text faint (10 px labels) | #8f949c | BM, SM (`--ink-faint`) |
| Accent red | #d92e3a | all three: knob pointers, LEDs, playhead, active states, hazard strip, logo text-shadow |
| Dark red | #8f1a28 | BM, SM `--accent-2`; MX `--neve-red` |
| Amber | #e0ab52 | BM, SM, MX: status text, solo, legend, VU mid |
| Start / mixdown button gradient | #ff6a45 to #d92e3a | BM, SM start; MX mixdown (border #c72c14) |
| Radial glow at top | rgba(217,46,58,.10) | BM, SM, MX |
| Rivets / hazard strip / 0.8 s power-on flicker | n/a | BM, SM described; MX has cheeks and hazard line |

Channel plate shades (BM): #35383d, #2e3034, #282a2d, #212325, #1b1c1e; SM rows: #3a3d42, #2e3034, #25272a, piano-roll plate #35383d. Track colours in BM: kick and snare #d92e3a, hi-hat, cymbal, perc #b9bec5.

Fonts: **Michroma** (400, Google Fonts, fallback 'Arial Narrow', sans-serif) for logo, buttons, channel and FX names, LCDs; **Barlow** 400/500/600/700 (fallback 'Helvetica Neue', Arial) for body, labels and values. Sizes mostly 9-12 px, logo `clamp(22px, 3.6vw, 34px)` (BM) or `clamp(20px, 3.6vw, 34px)` (SM) and 22 px (MX); tabular numbers on readouts. Loaded from `fonts.googleapis.com` in all three (the only external request).

Layout pattern: a centred "unit" card, `.stage` width 1180 px in BM, max-width 1180 px in SM, 16 px radius, 4 px red hazard strip on top, four 9 px rivets, header row (54 px PNG logo as base64 data URI, title with red offset shadow, subtitle, sync box with LED, status and countdown), then stacked "racks" of plates. Each plate is a flex or grid row with a vertical rotated name rail, divider and controls. BM: MASTER | FILTER, REVERB | DELAY, then the 5-channel sequencer rack. SM: MASTER | OSC, full-width piano roll, FILTER | ENVELOPE, REVERB | DELAY. MX: brand row, flex row of 136 px channel strips, ADD TRACK tile, 300 px master module, full-width timeline.

Components:

- **Knob**: pure CSS/DOM, 52 px dial (BM, SM; a 34 px small variant exists in CSS) with radial gradients, 3 px red pointer rotated -135 to +135 degrees, centre cap, label and value; `role=slider` with aria values in BM and SM. MX knob is 34 px (26 px small), pointer 2x12 px #d92e3a, no role or keyboard.
- **Fader**: native `input[type=range]` restyled with 14 px red thumb (BM per-channel faders) or rotated -90 degrees with a 44x16 px cream cap (MX, 160x30 px, dB scale labels positioned by JS).
- **LED**: CSS circles, 9 px per channel (colour var(--c)), `.fx-led` 8 px, `.sync-led` 11 px with `steps(1)` blink while locking; BM `.fx-led` never lit by code (l.522, 1503, 1533).
- **VU**: 10 span segments per channel, 22x84 px, green #4caf6d (1-6), amber (7-8), red (9-10), off #15171a (BM, SM); MX uses a canvas 14x160 per strip and 12x160 per master channel with gradient #5c8f5c, #d9a441 (0.78), #e2372f (0.92).
- **Step pad**: BM `button.pad` square with fill rising from the bottom (0 / 48 % / 100 %), `.playhead` 2 px ink outline, `.hit` brightness 1.9 for 130 ms. SM has no DOM steps; the grid is three hand-drawn canvases (roll, ruler, velocity), `devicePixelRatio` scaled.
- **Buttons**: `.tbtn` transport, `.stepbtn` +/-, mute/solo 34 px squares (red or amber when active), `.pill-btn`, `.wave-btn` with inline SVG wave icons (SM, the only SVG); MX gradient buttons with hard bottom shadow.
- **LCD**: CSS div with Michroma red or amber text and glow (BM, SM); MX uses an amber status line and time display.
- **Accessibility**: BM and SM honour `prefers-reduced-motion`, give knobs aria values and use focus-visible outlines in accent red (BM l.294-296).

### How CP5 differs

Single-accent neon on near-black, system font stack, native range sliders restyled with CSS only (no knobs, no faders), step "buttons" as div cells with embedded sliders, no external requests, no logo image.

| File | Base / panel | Accent | Fonts | Layout |
|---|---|---|---|---|
| D4 | #0b0b0e, panel #141418 to #1e1e24 | #39ff14 (fluogreen), text #e7ffe7, muted #9fb09f | system stack | `.wrap` 1100 px, 2-column top grid, rows of 190 px + 16 cells |
| S3 | #0b0b0d to #121216, card #1b1b22 to #17171d | #ff5f00 (fluo orange), text #e6e6ea, muted #9a9aa2 | Inter, Segoe UI, Roboto (Inter not loaded) | card 1100 px, 4 panels in a grid, 16 step columns |
| SL | #0b0b0b, panel #151515 to #1e1e1e | #eaff00 (fluogeel), text #e7e7e7, muted #9aa0a6 | system stack, monospace for kbd chip | card 1000 px, four flex panels |
| TH | #0b1020, panel #101731, border #1b254a | #6bc4ff (declared, used as rgba only), text #e8eefc | system stack | CSS grid, `min(980px, 96vw)`, 2 columns collapsing at 860 px |
| MT | #000, panel #1a1a1a | #ff00ff magenta, accent-color #39ff14 on sliders | Arial | single column, two cards 80 % wide |

### Consistent and conflicting values

Consistent across the RABBIT files: #131415, #1c1d1f/#101112, #1a1c1e, #f0f0ee, #b9bec5, #d92e3a, #8f1a28 (name differs in MX), #e0ab52, the red-to-orange start gradient, Michroma + Barlow, the Google Fonts URL, the base64 PNG logo.

Conflicts and drift:

- MX uses its own token names (`--wood-*`, `--metal-*`, `--cream`, `--neve-red`) for what BM and SM call `--bg`, `--chassis-*`, `--ink`, `--accent`; MX strip body #34373b and border #7a7f87 versus BM/SM #3a3d42 and #585c62.
- Green differs: #4caf6d (BM, SM VU), #5c8f5c (MX VU), #6da06d (MX play button).
- Accent hard-coded outside the tokens: BM `getComputedColor` repeats #d92e3a and #b9bec5 (l.1492-1495); SM has `const ACC` (l.473) and rgba literals; MX uses rgba(194,31,43) shadows and rgba(217,164,65) loop region that do not match #d92e3a and #e0ab52.
- Unused tokens: MX `--panel`, `--screw`; CP5 files each declare unused variables (D4 `--accent-dim`, `--danger`, `--line`; S3 `--accent`, `--good`, `--bad`; SL `--accent-dim`, `--warning`, `--ok`, `--danger`; TH `--ok`, `--warn`, `--err`).
- CP5 accents conflict by design (five different hues); CP5 hex values are repeated as literals in CSS and canvas code (D4: only 6 tokens).
- Layout widths conflict: 1180 px (BM, SM), horizontal scroll (MX), 1100/1100/1000/980 px (CP5). Only TH and SM have breakpoints.
- Dark only in every file; none handles `prefers-color-scheme`.

## 6. Engineering weaknesses

### Timing and scheduling

- `setInterval` scheduler with 0.12 s lookahead (0.25 s in SL) breaks in hidden tabs, exactly the multi-tab use case: D4, S3, SL, BM, SM. No catch-up guard, so a stall produces a burst of past-time notes: BM, SM, SL.
- MT uses `setInterval` plus `osc.start()` without a time argument (l.132-133, 122), no drift correction.
- MX loop wrap stops and recreates all sources from the main thread with a 60 ms delay (l.641-644, 566), an audible gap; a double restart is possible (l.574, 599-610).
- D4 step 0 plays about 25 ms late (l.431-432); BM and SM visuals ignore output latency and lead the audio; D4 highlight is drawn at schedule time, up to 120 ms early (l.397).
- rAF loops never cancelled: D4, S3 (multiplied by repeated Start), SL, BM (two), SM, MX (one per strip plus master plus timeline). In SM, `visualQueue` grows unbounded in hidden tabs and the UI sticks on "LOCKING"; in BM, `visualQueue` grows too.

### Synchronisation

- One-shot wall-clock anchor, no resync, no latency compensation: D4, S3, SL, BM, SM (section 4).
- S3 starts immediately and "synced" at a random offset when `mod == 0` (l.532-536).
- Tempo local to each page, no shared transport: all syncing files.
- BM and SM race on a suspended `AudioContext` when anchoring (l.1673-1677; SM l.598 vs l.1201). SM "LOCKED @" shows the planned boundary. BM header note (l.583) and SM text (l.332) over-promise.

### Start/stop correctness

- Pending start timeout not cancelled by Stop: D4 (l.428, duplicate schedulers, audio cannot be stopped), S3 (l.544, double Start gives two chains, `isRunning` write-only), SL (l.421).
- Stop does not silence scheduled voices: D4, S3, SL, BM (l.1700-1713, can flip status back to RUNNING, l.1657-1663), SM (tails up to about 7 s), MX (STOP loses exact position, no pause).
- SM: `Space` in D4 (l.535) has no `e.repeat` check and blocks Space on focused controls.

### Audio engineering

- No limiter or compressor in D4, S3, SL, BM, MX (SM has the only compressor, -10 dB, 8:1); gains can exceed 1.0 (D4 up to 1.8; MX up to +6 dB per strip and master; hard clipping in the WAV encoder, l.1307). Meters that hide clipping: D4 (RMS x1.9), SL (CLIP tests RMS, not peak), BM (RMS x3.2).
- Zipper noise from direct `.value` writes: D4 (l.520-531), S3 (l.566-573). S3 `setTargetAtTime` conflicts with slider writes (l.499-500).
- Reverb impulse response rebuilt synchronously on every knob event, random each time, swapping `convolver.buffer` live (click, jank): SL (up to 6 s), BM (up to 4 s), SM (up to 6 s), MX (debounced).
- Envelope errors: S3 `offTime` expression always equals `t0 + noteDur` (l.509), so A and D are ignored and `setValueAtTime` can land inside ramps (l.510); SL has the same overlap when A+D exceeds the note length (l.471-474).
- Biquad Q for LP/HP is in dB in Web Audio, so "Res" 100 % produces an about 18 dB peak: BM (l.834), SM (l.495).
- D4: the same 1 s noise buffer from offset 0 for every snare and hi-hat hit, cut at 1 s when decay exceeds it; default lowpass 4000 Hz attenuates most of the hi-hat.
- BM: samples truncated at decay+0.05 s (l.1031); SM: comment says two oscillators, code has one (l.602, 621-622); overlapping same-pitch notes stack (l.1068-1072).
- S3: degrees above scale length clamp instead of wrapping, so 7-note scales repeat degrees 6-8 (l.463).
- MT: no envelope, audible pops; AudioContext created at load, `resume()` never called; `60000/0` gives Infinity.
- TH: unmirrored video makes X-to-volume feel inverted; no min/max validation on the frequency range; `found` true on almost any frame; oscillator runs forever at gain 0.
- Meter limits: BM `smoothingTimeConstant` has no effect on `getByteTimeDomainData`; MX 256-sample peak per frame misses true peaks.

### State and architecture

- One IIFE or global script with mixed model, audio nodes and DOM, DOM as source of truth, no store, no modules, no types, no tests: all eight. Specific cases: BM `CHANNELS` mixes steps and settings with `gainNode`, `rowEl`, `padEls` (l.740-753); SM knob value duplicated in closure and `S` (l.663, 729-733); MX `set()`/`get()` unused (l.760), hidden coupling `playBtn`/`stopAll`.
- MX builds the live graph (l.482-503) and the offline graph (l.1199-1237) twice by hand; MX window listeners per knob never removed (about 260 with 32 tracks, l.752-757).
- Duplicated code: D4 DOM queries per step (l.543, 552, 565-566) and hard-coded channel indices [0,1,2] (l.557, 560); S3 nine near-identical readout listeners; BM `hpCutToFreq`/`lpCutToFreq` identical, copy-pasted FX builders; SM 19 near-identical `makeKnob` calls.
- Dead code: unused CSS variables and rules in every file; SM `defVel`, `.vel-new`, `drag.moved`; BM `wallMsForAudioTime`, `cfg.tick`; SL `sliceMs`/`jitterMs` globals and empty jitter handler (l.397); TH `recalibrate()` only alerts (l.354-360).
- No persistence in any file; reload loses everything including loaded samples.
- Pre-audio control bugs: D4 channel volume ignored before first Start (l.244, 370); SL filter, wet and reverb-size handlers throw until a file is loaded (l.398-402) and `ensureAudio` hard-codes 1200 Hz, Q 0.8, wet 0.18 (l.258, 261).
- Unguarded or lost errors: SL `decodeAudioData` with no try/catch (status stuck on "Laden..."); SM `new AudioContext` without try/catch; TH raw `alert()` on error.
- SM: voice limit silently drops notes (l.605). MX: AudioContext created at load before a gesture (l.386). BM: reselecting the same file after load does nothing.

### Markup and compatibility

- BM: no doctype, html, head or body (starts with `<title>`, quirks mode, possible mojibake at l.1301 and l.1412); SM: no `</head>` or `<body>` (l.311-313).
- Non-standard CSS: S3 vertical sliders use `writing-mode:bt-lr` and `-webkit-appearance:slider-vertical` (Chromium/Safari), Firefox track ignores `--fill`; D4 and S3 `.vel` slider has only `-webkit` rules (D4 l.106-120).
- Google Fonts request at load (BM, SM, MX): privacy (GDPR) and offline fallback to Arial Narrow; MX states this explicitly.

### Accessibility and responsiveness

- Controls without labels or `aria-label`: D4 (only `#tempo` has `for`/`id`), S3 (16 selects, 16 sliders), TH (labels without `for`), MX (rotated range inputs unnamed).
- MX knobs: plain divs without role, tabindex, keyboard or wheel; BM knobs lack `aria-valuetext`, wheel handler always `preventDefault` (l.1283), right-click starts a drag (l.1260); SM roll is pointer-only, so its keyboard handler (l.1083-1094) is unreachable for keyboard-only users.
- Status and countdown not `aria-live`: D4, S3, SL, BM, SM, MT (slot display).
- 10 px text in #8f949c on #3a3d42 about 3.5:1 contrast (BM, SM, report estimates).
- Not responsive: D4, S3, SL, MT, BM (fixed 1180 px stage), MX (more than 4300 px wide at 32 strips); SM and TH have breakpoints.

### Per-file defects that illustrate the workshop point

S3 and BM state it directly: S3 ("the architecture limit the workshop wants to show": patterns cannot be shared between tabs, only the start time); MT's title mixes two unrelated tools, "evidence of prompt-by-prompt accretion".

## 7. Open questions

1. **Meaning of "one application with multiple synchronised tabs"**: the reports show only start-time alignment from wall clock. Whether tabs should share transport, tempo, patterns (e.g. via server or BroadcastChannel) is not settled by any report; none of the files has inter-page communication.
2. **Target tempo behaviour**: should a tempo change propagate to all tabs, and should a late-joining tab start at the correct step? Every existing file resets `currentStep = 0` on start; no report covers a join-in-progress design.
3. **Whether a clock source other than `Date.now()` is wanted** (server time, NTP offset estimate): the reports note only that wall-clock agreement depends on OS clocks.
4. **Which instruments belong in scope**: MT (metronome plus slot machine) and TH (webcam theremin) are not sequencers and have no sync; the reports do not say whether they should be rebuilt, merged (metronome into transport) or dropped. TH needs a secure context for the camera.
5. **How CP5 instruments are mapped onto the RABBIT look**: five different CP5 accents exist (#39ff14, #ff5f00, #eaff00, #6bc4ff, #ff00ff) while the brief says "same design". The reports do not say whether a unified red/amber skin replaces them or per-instrument accents stay.
6. **Fate of CP5-only functions** (per-step sliders in D4, scale degrees in S3, random slice sampler): the reports do not rank them against the RABBIT equivalents (BM pads, SM piano roll).
7. **Unmeasured behaviour**: all drift magnitudes (tens of ppm, a few ms per minute), double-click toggle on D4 steps ("probably ends at 0", unverified), Firefox rendering of `.vel` and S3 tracks (unverified), Space double-toggle on a focused button (unverified). Needs a run in a browser, not a code read.
8. **Meaning of unlit or decorative UI**: BM FX LEDs never light; SM `defVel` has no control (a removed knob?); MX `updateBrandSub` writes a constant; whether the engineer should wire these or drop them is open.
9. **Audio behaviour to keep versus fix**: e.g. D4's quiet hi-hat under the default filter, MX "Class-A" naming with no modelling, SM's compressor settings. The reports describe them, not the intent.
10. **Sample rate, export format and limits**: MX renders at device sample rate, 16-bit, no dither, no progress or cancel; whether a fixed rate or other formats are wanted is not stated. File size or duration limits for sample loading are absent in SL, BM, MX.
11. **Language**: D4, S3, SL, TH, MT, MX use Dutch UI text (`lang="nl"`) with English labels mixed in; the reports do not say what language BM and SM should show or whether an i18n layer is wanted.
12. **Persistence and sharing** (save/load, URL state, undo): absent in all eight, so no report gives a format or scope; this is a product decision.
13. **Fonts**: Michroma and Barlow are loaded from Google in BM, SM, MX; whether to self-host in the containerised app is not decided by the reports.
