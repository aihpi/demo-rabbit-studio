# Shell and design system

## 1. Sources

This module is not an instrument. It collects what the three RABBIT files share and decides where they differ.

| Short name | Original file | Report | What this module takes from it |
|---|---|---|---|
| BM | RABBIT-BeatMachine_x.html | report_RABBIT-BeatMachine_x.json | header transport (start, stop, tempo, swing), knob, pad, LED, VU, tokens, fonts, buttons |
| SM | RABBIT-SynthMachine1_Full.html | report_RABBIT-SynthMachine1_Full.json | LCD, tempo reset value, breakpoints, knob double-click reset, transport button size, tokens |
| MX | RBBT-MixConsole1_Full.html | report_RBBT-MixConsole1_Full.json | vertical fader taper, status line, peak readout, tokens |

All three files use the Google Fonts stylesheet and the same token family under three different naming schemes (BM and SM: `--chassis-raised`, `--ink`, `--accent`; MX: `--metal-mid`, `--cream`, `--neve-red-bright`). The older CP5 files are not sources for this module. Their neon colours are dropped and not specified here.

Files created by this module under `01_frontend/`: `index.html`, `styles/tokens.css`, `styles/chassis.css` (plates, hazard strip, rivets, layout), `styles/components.css` (buttons, select), `fonts/` (woff2 files), `img/logo.png`, `ui/shell.js`, `ui/header.js`, `ui/tabs.js`, `ui/knob.js`, `ui/fader.js`, `ui/led.js`, `ui/vu.js`, `ui/pad.js`, `ui/lcd.js`, `audio/meters.js`, `fx/reverb.js`, `fx/delay.js`. There is no per-instrument CSS file, no `rabbit.css`, no `assets/` folder and no `shell/` folder. The folder layout is in docs/plan.md.

## 2. Purpose

The shell is the page that holds all instruments: a fixed header with the global transport and a master meter, a tab row that shows one LED per instrument, and a content area into which the active instrument mounts. The design system gives every instrument the same dark RABBIT hardware look through one token file, self-hosted fonts and six shared components, so that no instrument carries its own copy of a knob, fader, LED, meter, pad or LCD. The shell also fixes the accessibility defects that all three source files share. The smallest supported width is 640 px.

## 3. Feature parity

The shell does not implement instrument features. A row that says "stays in instrument spec" means the feature belongs to that instrument module and uses the shared components defined here. Rows that name a shared component say how the component changes.

### 3.1 BM (18 entries)

| Feature in the source | Source | In RABBIT Studio |
|---|---|---|
| 5 fixed channels KICK, SNARE, HI-HAT, CYMBAL, PERC with editable names (l.732-738, 1354-1366) | BM | Stays in beat instrument spec. Editable name uses `aria-labelledby` on the element so a renamed channel is announced correctly (fixes stale aria-labels, l.1314-1342). |
| 16 steps per channel as 4 groups of 4, ruler 1, 5, 9, 13, empty patterns (l.1104-1123, 733-737) | BM | Stays in beat spec. Step button is the shared `ui/pad.js`, 36 px cell, 4 px gap, 18 px between groups. |
| Step button with 3 levels off, full (1.0), half (0.5), aria-pressed false/mixed/true (l.1461, 1469) | BM | `ui/pad.js` takes a value from 0 to 1, shown as fill height. A click cycles 0, 1, 0.5, 0. `aria-pressed` is false for 0, true for 1 and mixed otherwise. The synth piano roll is canvas and stays in synth spec. |
| Per-channel RND (35 % on, level 1 or 2 at 50/50) and CLR (l.1330-1350) | BM | Stays in beat spec. Button style is `.btn--pill` from this module. |
| Per-channel MUTE and SOLO, isSilent rule, silent channel at opacity .5, evaluated at scheduling time so up to 0.12 s late (l.1037-1042, 1308-1328, 1650) | BM | Stay in beat spec for the drum channels and the step pad (the beat instrument keeps its own isSilent rule). Mute and solo of the whole instrument live on the Console strip. Both use `.btn--toggle` (34 px square, red when mute is active, amber when solo is active). |
| Per-channel VOL knob (0 to 100 %, defaults 82, 78, 60, 55, 70), PAN knob (-100 to +100, default 0, C/nL/nR, double-click reset), faders DECAY, PITCH (-12 to +12 semitones), RVB, DLY (defaults per channel) | BM | All of these are per drum voice and stay in beat spec. They use `ui/knob.js` (PAN with `reset: 0` and a C/L/R `format`) and `ui/fader.js`. The Console strip adds a separate instrument-level gain and pan on top. |
| Sample loading per channel, 16-character file name, x clears, 'LOAD ERROR' (l.1405-1438) | BM | Stays in beat spec. The file goes to the backend and is referenced by the key `sampleId`. The error text is shown on the channel in the beat spec; an instrument cannot write the header status LCD. |
| Synth voices kick, snare, hi-hat, cymbal, perc | BM | Stays in beat spec. |
| Transport START and STOP, icon-only glyphs, START disabled while playing, STOP disabled while stopped, no pause (l.208, 1672-1713) | BM | Moves to the global header. Same enable rule. Glyphs become inline SVG with `aria-label` "Start" and "Stop". No pause is added. |
| Tempo 40 to 300 BPM, default 120, knob plus +/- buttons (+/-1, Shift +/-5), retimes the delay (l.1583-1595, 861-864) | BM | Moves to the global header. Knob reset value 120 (SM wins over BM, which has none). The range and default are owned by docs/plan.md, section Transport. `fx/delay.js` retimes itself through `transport.subscribe`; instruments do not. |
| Swing 0 to 70 %, default 0, even steps lengthened and odd steps shortened by stepDur*swing/100*0.5 (l.1077-1078, 1597-1600) | BM | Moves to the global header. The transport (docs/plan.md, section Transport) applies the same formula. Both BM and SM use the identical formula, so no conflict. |
| Global filter HPF and LPF with Cut and Res, cut mapping 20*(20000/20)^(v/100) Hz, Q lerp 0.7 to 18 (l.832-834, 1611-1642) | BM | Stays in beat spec. Note: SM maps cut as 20*1000^(v/100), which is the same curve. Both use the shared knob. |
| Reverb send DECAY 0.2 to 4.0 s, SIZE, EQ, VOLUME -24 to +6 dB, IR regenerated on change (l.839-853) | BM | Stays in beat spec; the unit uses `fx/reverb.js` (debounced rebuild, see Defects fixed). The decay range is a parameter of the caller (BM 4 s, SM 6 s, MX 8 s). |
| Delay send TIME select 11 divisions, FEEDBACK 0 to 95 %, EQ, VOLUME (l.813-825, 1538-1567) | BM | Stays in beat spec; uses `fx/delay.js`. The list of divisions is a parameter of the caller (BM 11, SM 10). |
| Stereo VU, 10 segments per side, RMS of an AnalyserNode, segments 1 to 6 green, 7 to 8 amber, 9 to 10 red (l.769-795) | BM | `ui/vu.js` keeps the 10 segments and the colours; the caller passes the dB range. The meter loop moves into `audio/meters.js`; `ui/vu.js` registers on mount and has no loop of its own. The master VU moves to the global header; per-instrument meters belong to the Console strip. |
| Sync box with LED, STOPPED / LOCKING... / RUNNING, countdown 'STARTS IN x.xs', 'LOCKED @ HH:MM:SS.mmm' (l.578-585, 1662, 1666-1670) | BM | Wall-clock start: removed by decision. Countdown: removed by decision. The sync box becomes the header status LCD with STOPPED and RUNNING only. |
| Playhead outline on ruler and pads, 130 ms hit flash (l.1649-1655) | BM | `ui/pad.js` keeps `playhead` (2 px ink outline) and `hit` (brightness 1.9, 130 ms). The beat instrument writes `{ stepIndex, time }` into its ring buffer from `onStep`, and the mounted interface sets both classes from it in the shared animation loop. |
| Decorative power-on flicker, hazard strip, rivets, blind-shadow bars, footer legend (l.51-61, 717-718) | BM | Kept on the shell chassis (see Kept on purpose). The footer legend text is replaced by one shell legend line. |

### 3.2 SM (21 entries)

| Feature in the source | Source | In RABBIT Studio |
|---|---|---|
| Piano roll 16 steps by 36 rows, MIDI 48 to 83, canvas (l.469-471, 1161) | SM | Stays in synth spec. Canvas colours read the CSS tokens at start (no hex literals in JS). |
| Demo pattern at start, 21 notes, LCD 'DEMO PATTERN' (l.526-534) | SM | Stays in synth spec. The LCD is the shared `ui/lcd.js`. |
| Add note by double-click, len 1, vel 100 (l.1010-1035) | SM | Stays in synth spec. |
| Delete note by double-click or Delete or Backspace (l.1024-1027, 1086) | SM | Stays in synth spec. |
| Select note, drag move, drag right edge resize, collisions allowed (l.1001, 1028-1072) | SM | Stays in synth spec. |
| Keyboard on the focused roll, selected note only (l.1083-1094) | SM | Stays in synth spec. |
| Piano-key strip click previews 0.22 s, key flash 180 ms (l.1015-1018) | SM | Stays in synth spec. |
| Velocity lane click or drag, 1 to 127 (l.1096-1127) | SM | Stays in synth spec. |
| RND (A-minor pentatonic) and CLR (l.1129-1152) | SM | Stays in synth spec. Buttons use `.btn--pill`. |
| Transport Start and Stop with 4 s wall-clock wait, LOCKING..., countdown, 'LOCKED @' (l.1199-1220) | SM | Moves to the global header. Wall-clock start: removed by decision. Countdown: removed by decision. |
| Playback 16th-note steps, pattern 240/bpm s, loops forever (l.1171) | SM | Moves to the global transport; the synth gets `onStep(stepIndex, time, stepDuration)`. |
| Swing 0 to 70 % (l.1178-1179) | SM | Moves to the global header (same formula as BM). |
| Polyphony cap 16, 'n/16' LCD (l.469, 604-605, 1231) | SM | Stays in synth spec. LCD is `ui/lcd.js`. |
| Oscillator waveform sine, saw, tri, square, default saw, level factors, preview A4 (l.474) | SM | Stays in synth spec. The 56 by 40 px wave button with inline SVG is a synth-local `.btn--toggle` variant; its stroke uses `currentColor`. |
| Octave shift -2 to +2 (l.766-770, 1173) | SM | Stays in synth spec. |
| Detune -50 to +50 cents (l.622) | SM | Stays in synth spec. |
| ADSR with envelope canvas preview (l.800-830) | SM | Stays in synth spec. |
| Two filters in series HPF then LPF | SM | Stays in synth spec. |
| Reverb with noise IR (decay 0.2 to 6 s) and delay with 10 tempo-synced divisions | SM | Stays in synth spec; uses `fx/reverb.js` and `fx/delay.js`. |
| Master volume -30 to +6 dB (default -4 dB, gain 0.631, reset -4), compressor, stereo L/R VU, 10 segments (l.753-756) | SM | Master volume: moves to the Console master fader. The synth gets the Console strip volume with the strip default of section 6. The -4 dB of the original stays as a fixed trim of gain 0.631 inside the synth chain, as the synth spec says. The compressor stays in synth spec (it is part of the synth sound). VU: `ui/vu.js`; master meter in the header. |
| Knob interaction: vertical drag 150 px, touch, wheel, arrows, Home/End, double-click reset, live delay milliseconds (l.661-734) | SM | Becomes the behaviour of `ui/knob.js` (see Components). SM wins over MX for keyboard, MX wins for "double-click resets everything"; `reset: null` makes a double-click do nothing. Live millisecond text is a `format` function passed per knob. |

### 3.3 MX (19 entries)

| Feature in the source | Source | In RABBIT Studio |
|---|---|---|
| Channel strips, 6 at start, ADD TRACK up to 32, no remove (l.345-346, 944-961) | MX | Stays in Console spec. `audio/mixer.js` builds one strip per instrument and per WAV track; the Console shows them and builds none. |
| Strip header 'CH 01', editable name, maxLength 14 (l.769-774) | MX | Stays in Console spec. |
| LOAD WAV with hidden file input, decodeAudioData, 'LOADING...', error text (l.781-800) | MX | Stays in Console spec. File goes to the backend and is referenced by id. Message text is English and appears in the status LCD. |
| CLEAR button per strip (l.801-811) | MX | Stays in Console spec. |
| Per-strip EQ: LO-CUT 75 Hz (default off), LOW shelf 120 Hz, MID peaking Q 1, HI shelf 8 kHz, each -15 to +15 dB, default 0 (l.817-859) | MX | Stays in Console spec; the three EQ knobs use `ui/knob.js` at size `sm` (26 px, MX `knob-sm`). |
| MID frequency dropdown 250, 500, 1k (default), 2.5k, 5k, 8k Hz (l.847) | MX | Stays in Console spec. Select uses the shared `.select` class (recess well, `--stroke`). |
| PAN knob -1 to +1, 'C' within 0.03 (l.661-664, 862-866) | MX | Stays in Console spec; size `md` (34 px). `ui/knob.js` `format` and `reset` options cover it. |
| REV SEND knob 0 to 1 (l.867-871) | MX | Stays in Console spec; size `md`. |
| Vertical volume fader with dB taper, 0.75 = 0 dB, top +6 dB, floor -60 dB, true silence at 0.001, default gain 0.8 (about -1.9 dB) (l.347-372, 693) | MX | `ui/fader.js` option `taper: "db"` keeps these exact numbers. BM faders keep `taper: "linear"`. The strip default gain (0.8) is in section 6. |
| Per-strip peak meter canvas (14 by 160 px, peak not RMS, -60 to +6 dB, white 0 dB line) with peak hold, click resets (l.878-890, 914-922) | MX | `ui/vu.js` (10 DOM segments, RMS) replaces the canvas meter by decision of the target description. The peak-hold readout stays as option `peak: true`, which adds a numeric dB `ui/lcd.js` readout that resets on click. |
| M and S per strip, solo overrides mute, several solos allowed (l.519-525, 906-912) | MX | Stays in Console spec. Buttons use `.btn--toggle`. |
| Reverb module with DECAY 0.2 to 8 s (default 2.2), SIZE 0 to 1 (0.5), LO-CUT 20 to 1000 Hz (150), HI-CUT 1500 to 18000 Hz (9000) (l.445-472, 975-985) | MX | Stays in Console spec; uses `fx/reverb.js`. |
| Master bus REV RTN and MASTER faders (default 1.0 each), L/R meters (l.1008-1042) | MX | Master fader stays in Console spec. The master L/R meter also appears in the header (same analysers). |
| Transport rewind, PLAY/STOP toggle, STOP; no pause position (l.1046-1053, 584-588) | MX | Moves to the global header: start and stop of the global transport. Rewind: stays in Console spec (timeline). The Console has no second Play button; its audio-file playback follows `onStart()` and `onStop()`. The green play state is dropped; running is shown by the header LED. |
| LOOP toggle (default off), loop region (l.1056-1065, 641-644) | MX | Stays in Console spec. |
| Timeline bar with playhead, loop handles (minimum loop 0.05 s), click seek, time display (l.1089-1157, 614-619) | MX | Stays in Console spec. |
| Checkbox 'alleen loop-selectie mixdownen' (default unchecked) and mixdown button (l.1067-1077) | MX | Stays in Console spec. Label becomes English. Buttons use `.btn--primary` (the red gradient). |
| Offline mixdown to 16-bit WAV, file name mixdown_YYYY-MM-DD-HH-MM-SS.wav (l.1163-1311) | MX | Stays in Console spec. It covers every switched-on instrument except the Theremin (stage 9). |
| Status line in amber with Dutch messages (l.612, 1079) | MX | Moves to the global header status LCD, in English. |

### 3.4 Controls that no feature row names

These entries of the "controls" arrays carry details that the rows above do not state. Each one has an owner.

| Control in the source | Source | In RABBIT Studio |
|---|---|---|
| No master volume, fixed masterGain 0.9 (l.883); no tap tempo, no pattern save, no export | BM | The fixed 0.9 gain is part of the beat sound and stays in beat spec. Save and load come from the project backend. |
| Step pads: no drag painting, no modifiers, normal button activation only | BM | `ui/pad.js` adds none; Enter and Space cycle the level. |
| Start (play triangle, enabled initially) and Stop (square, disabled initially) | SM | Global header buttons, same initial states. |
| Roll info LCD, read-only, 230 px wide; voices LCD 'n/16'; octave -/+ buttons with aria-live LCD | SM | Stay in synth spec on `ui/lcd.js`. |
| Wave buttons, detune, filter, envelope, reverb and delay knobs with their defaults and reset values (l.753-859) | SM | Stay in synth spec; `ui/knob.js` takes `default` and `reset` per knob. |
| Mute, solo (default off), LO-CUT (default off), loop (default off), render checkbox (default unchecked), loop handles default 0 and total duration | MX | Stay in Console spec with `.btn--toggle`. |
| Track name input, LOAD WAV and clear, ADD TRACK tile, peak reset by click | MX | Stay in Console spec. |
| Knob drag 140 px, sweep -135 to +135 deg, double-click resets to the initial value, no keyboard, no wheel | MX | `ui/knob.js` uses 150 px (BM, SM), the same sweep, double-click reset and keyboard (see Components). |

## 4. Controls

### 4.1 Shell controls

| Control | Component type | Range | Default | Source |
|---|---|---|---|---|
| Start | `.btn--transport` with SVG glyph, 38 px square | none | enabled while stopped | BM l.1672-1713; size from SM `.tbtn` 38 px |
| Stop | `.btn--transport` with SVG glyph, 38 px square | none | disabled while stopped | BM l.1672-1713 |
| Tempo | `ui/knob.js`, size `lg` (52 px), unit BPM | 40 to 300, step 1 | 120, reset 120 | BM l.1592-1595, SM l.737-740 |
| Tempo minus and plus | `.btn--step`, 24 by 22 px | step 1, Shift step 5, clamped to 40 and 300 | none | BM l.1583-1590, SM l.747-748 and `.stepbtn` |
| Swing | `ui/knob.js`, size `lg` | 0 to 70 %, step 1 | 0, reset 0 | BM l.1597-1600, SM l.749-752 |
| Status LCD | `ui/lcd.js`, tone red, `aria-live="polite"` | text | STOPPED | BM l.578-585, MX l.612 |
| Master meter | `ui/vu.js`, two rows of 10 segments, horizontal | -60 to +6 dB, lit = round(level * 10) | idle, none lit | BM l.769-795, SM l.642-658 |
| Run LED | `ui/led.js`, 11 px | off or on | off | BM `.sync-led` |
| Instrument tab (6) | `button role="tab"` | one active; interface state, not in the project | beat | target description |
| Instrument on/off switch (5, none on the Console) | `button role="switch"` with `ui/led.js`, 9 px; stored as `on.<id>` in the project | off or on | beat on, the others off (proposal, see Open questions) | target description |

### 4.2 Tokens

All tokens live in `01_frontend/styles/tokens.css` under `:root`. One name per colour. No file other than `tokens.css` may contain a colour value in any notation (hex, `rgb()`, `rgba()`, `hsl()`); `chassis.css`, `components.css` and the JS canvas code read tokens (JS through `getComputedStyle(document.documentElement)`). The "Winner" column says which source value is kept.

| Token | Value | Use | Conflicts resolved, winner |
|---|---|---|---|
| `--bg` | #131415 | page base | BM `--bg`, SM `--bg`, MX `--wood-dark` are the same value. |
| `--floor-a`, `--floor-b` | #1c1d1f, #101112 | 2 px scanline stripes on body | same in BM, SM, MX. |
| `--recess` | #1a1c1e | wells: pads, selects, LCD surround, buttons end colour | BM `--chassis` and `--chassis-recess`, MX `--metal-dark`, `--screw` are all #1a1c1e. One name `--recess` wins. |
| `--plate-1` | #3a3d42 | default plate | BM and SM `--chassis-raised`. MX `--metal-mid` #34373b loses as a plate colour; its use as button gradient start is `--btn-top`. |
| `--plate-2` | #35383d | second plate row, piano roll plate | BM row 2 and SM roll plate agree. |
| `--plate-3` | #2e3034 | third row | BM and SM agree. |
| `--plate-4` | #282a2d | fourth row | BM #282a2d wins over SM `.lowest` #25272a (3 levels apart, not worth two tokens). |
| `--plate-5`, `--plate-6` | #212325, #1b1c1e | rows 5 and 6 of the beat channels | BM only. |
| `--unit-top`, `--unit-bottom` | #202226, #101112 | chassis gradient | BM and SM agree. |
| `--stroke` | #585c62 | borders | BM, SM. |
| `--stroke-bright` | #82868d | hover borders, button borders | BM and SM #82868d win over MX `--metal-light` #7a7f87. |
| `--ink` | #f0f0ee | main text | BM, SM `--ink`, MX `--cream`. |
| `--ink-dim` | #b9bec5 | labels on plates | BM, SM, MX `--cream-dim`. |
| `--ink-faint` | #8f949c | hints; allowed only on `--recess`, `--floor-a`, `--floor-b` | Same value everywhere. Restricted because about 3.5:1 on `--plate-1` fails AA (estimate by the BM and SM reports); about 5.6:1 on `--recess` (estimate, to be checked by the contrast test). |
| `--accent` | #d92e3a | red accent, playhead, pointers, active buttons, VU 9 to 10 | BM and SM `--accent`, MX `--neve-red-bright`. MX meter red #e2372f and rgba(194,31,43) shadows lose; shadows are `color-mix(in srgb, var(--accent) N%, transparent)`. |
| `--accent-dim` | #8f1a28 | idle LED, start-button border, loop handles | BM and SM `--accent-2`, MX `--neve-red`. Name `--accent-dim` wins. MX mixdown border #c72c14 loses. |
| `--accent-hot` | #ff6a45 | start and primary button gradient top (to `--accent`) | BM, SM, MX agree. |
| `--on-accent` | #1a0d08 | text on red and amber buttons | BM, SM #1a0d08. MX solo text #231a06 loses. |
| `--amber` | #e0ab52 | solo, amber LCD, legend, VU 7 to 8 | All three. MX meter amber #d9a441 and loop-region rgba(217,164,65) lose. |
| `--amber-hot` | #f0c15a | solo gradient top | MX only. |
| `--ok` | #4caf6d | VU segments 1 to 6 | BM and SM #4caf6d win. MX meter #5c8f5c and play-button green #6da06d are dropped; MX has no green button in the header. |
| `--vu-off` | #15171a | unlit VU segment | BM and SM. |
| `--led-off` | #33363a | unlit status LED | BM and SM. |
| `--lcd` | #0c0a0a | LCD well | SM #0c0a0a wins over the MX meter canvas #0c0c0c. |
| `--black` | #000 | knob border, LCD border, fader well border | All three use plain black; one token keeps the no-hex rule. |
| `--btn-top` | #34373b | button gradient start (to `--recess`) | BM and SM `.tbtn` and MX button start colour agree. |
| `--well-a`, `--well-b` | #0a0a0a, #1c1c1c | dark well gradient of the vertical fader track | MX only. |
| `--knob-hi`, `--knob-lo` | #4a4a4a, #0d0d0d | knob body radial gradient | BM and SM; MX knob (#3a3a3a start) is similar and loses. |
| `--knob-mid` | #3a3a3a | parameter fader track gradient start (to `--knob-lo`) | BM `.pslider`. |
| `--rivet-hi`, `--rivet-mid` | #cfd2d6, #5a5e64 | rivet gradient (ends in `--recess`) | BM and SM. |
| `--cap-a`, `--cap-b`, `--cap-c` | #e6dcc0, #b9ac86, #8c7f5c | vertical channel-fader cap | MX only. Kept because the console fader cap is part of its look. |
| `--track-red` | alias of `--accent` | track colour | BM `--c-kick`, `--c-snare`. |
| `--track-grey` | alias of `--ink-dim` | track colour | BM `--c-hihat`, `--c-cymbal`, `--c-perc`. |
| `--r-unit`, `--r-btn`, `--r-pad`, `--r-lcd` | 16px, 7px, 5px, 6px | radii | BM and SM. |
| `--specular` | rgba(255,255,255,.38) | knob highlight | BM and SM. Kept as a token so that no component file holds a colour value. |
| `--shadow` | 0 30px 60px -20px rgba(0,0,0,.75), 0 2px 0 rgba(255,255,255,.03) inset | chassis shadow | BM. |

Dropped on purpose: MX `--wood-light` #4a4d52, `--wood-mid` #232527 and the wooden cheeks (not part of the RABBIT chassis of the three newest files; the cheeks are not rebuilt), MX `--panel` and `--screw` (declared, unused), the CP5 neon colours.

### 4.3 Typography

Fonts are served from the container as files under `01_frontend/fonts/`: `michroma-400.woff2`, `barlow-400.woff2`, `barlow-500.woff2`, `barlow-600.woff2`, `barlow-700.woff2`. The CSS uses `@font-face` with `font-display: swap` and `src: url("../fonts/....woff2") format("woff2")`. There is no `fonts.googleapis.com` or `fonts.gstatic.com` request anywhere. `index.html` preloads `michroma-400.woff2` and `barlow-400.woff2`.

| Token | Value | Use |
|---|---|---|
| `--font-display` | 'Michroma', 'Arial Narrow', sans-serif | logo, buttons, LCD, plate names, strip numbers |
| `--font-body` | 'Barlow', 'Helvetica Neue', Arial, sans-serif | labels, values, status, selects |
| `--fs-logo` | clamp(22px, 3.6vw, 34px) | header logo; BM wins over SM clamp(20px, ...) and MX 22 px |
| `--fs-plate` | 13px | plate and channel names (BM 13, SM 13 to 14) |
| `--fs-lcd` | 12px | LCD, tab labels |
| `--fs-value` | 12px | knob and fader value readouts, tabular-nums |
| `--fs-label` | 10px | knob and fader labels, uppercase, letter-spacing 1px (MX) |

Sources use 9 px in some places (SM). The minimum here is 10 px; see Open questions about 11 px. Numeric readouts use `font-variant-numeric: tabular-nums` (BM).

### 4.4 Components

All components are plain ES modules that build DOM elements and return `{ el, set(value), destroy() }` or the variant below. No canvas, no SVG for knob, fader, LED, pad, LCD (BM, SM and MX all draw these with CSS and DOM). They write only to their own element and never create Web Audio nodes; `vu.js` receives an AnalyserNode.

| Component | Module | Size | Drawing technique | Keyboard and ARIA |
|---|---|---|---|---|
| Knob | `ui/knob.js` | `lg` 52 px, `md` 34 px, `sm` 26 px (BM 52 and 34, MX 34 and 26) | A `div` with two radial gradients, 3 px `--black` border, specular highlight from `--specular`, a 3 px red pointer rotated from -135 deg to +135 deg around its base (`transform-origin: 50% 100%`), 6 px cap. Label and value `span`s underneath. | `role="slider"`, `tabindex="0"`, `aria-valuemin`, `aria-valuemax`, `aria-valuenow`, `aria-valuetext` (formatted value with unit, for example "C", "3.3 kHz", "120 BPM"), `aria-labelledby` the label. Arrow Up and Right add one step, Arrow Down and Left subtract one step, Shift multiplies by 5, Home and End go to min and max. Wheel changes one step (Shift times 5) only while the knob has focus. Double-click sets the `reset` value; with `reset: null` it does nothing; with `reset` unset it restores `default`. The Enter key does nothing. Pointer: vertical drag with `setPointerCapture`, 150 px for full range, primary button only. |
| Fader, parameter (horizontal) | `ui/fader.js` | track 12 px high, 156 px slot, thumb 14 px round red with glow | Native `input[type=range]` restyled with `::-webkit-slider-thumb` and `::-moz-range-thumb` on a dark gradient track (BM). Label and value above. | Native slider keyboard (arrows, Home, End, Page keys). `aria-valuetext` set from the same `format` function. Label is a real `<label>`. |
| Fader, channel (vertical) | `ui/fader.js` | track 160 px high and 34 px wide, cap 44 by 16 px cream with dark centre line | Native `input[type=range]` with `writing-mode: vertical-lr; direction: rtl`, no CSS rotation. The dB scale marks (+6, 0, -20, -40, -inf) are absolutely positioned `div`s computed from the taper. Replaces the rotated range of MX. | Native slider keyboard; `aria-orientation="vertical"`; `aria-label` from the strip name (MX has none); `aria-valuetext` in dB with one decimal, "minus infinity dB" at silence. |
| Fader, compact vertical (step velocity) | `ui/fader.js` | height set by the caller (beat and synth step velocity), red round thumb, `taper: "linear"` | Same native range technique as the channel fader, without the dB scale marks and without the cream cap. | Native slider keyboard; `aria-orientation="vertical"`; `aria-label` "&lt;channel&gt; step &lt;n&gt; velocity". |
| Fader taper | `ui/fader.js` | none | `taper: "linear"` (BM) or `taper: "db"`: position 0 to 1, step 0.001; position 0.001 or less is gain 0; 0.75 is 0 dB; above 0.75, dB rises linearly to +6 at 1.0; below 0.75, dB falls linearly to -60 at position 0 (MX l.347-372). `posToGain` and `gainToPos` are exported and are exact inverses within one step. | none |
| LED | `ui/led.js` | 9 px (channel and tab, BM `.led`), 8 px (FX, BM `.fx-led`), 11 px (`lg`, run LED) | A `span` circle; off colour `--led-off` (status) or `--accent-dim` at 50 % (FX); on colour `--c` with 6 px glow (lg: 0 0 10px 2px and 0 0 22px 6px at 50 % accent). Blink state uses a 0.5 s `steps(1)` animation (BM `sync-pending`). | `aria-hidden="true"`; its meaning is always carried by a neighbouring text or switch (state is never colour alone, fixing MX mute and solo). |
| VU meter | `ui/vu.js` | 10 segments; vertical meter 22 by 84 px (BM; SM 20 by 76 px), horizontal 12 by 8 px per segment with 2 px gap in the header (new value of this spec) | A flex column or row of 10 `span.vu-seg`. Colours by `:nth-child`: 1 to 6 `--ok`, 7 to 8 `--amber`, 9 to 10 `--accent`; off `--vu-off`; glow 4 px. The caller passes `min` and `max` in dB (master meter and Console strips -60 to +6, Slicer -60 to 0). Level: `getByteTimeDomainData`, `rms = sqrt(mean(((byte - 128) / 128)^2))`, converted to dB and mapped linearly from `min` to `max` onto 0 to 1, lit = `round(level * 10)`. The meter registers with `audio/meters.js` on mount and deregisters on unmount. No ballistics (instant attack and release). Option `peak: true` adds a numeric peak-hold readout (`getFloatTimeDomainData` peak, -60 to +6 dB) that holds until clicked (MX). | `aria-hidden="true"`. The numeric peak readout (if present) is `role="status"` off and updated at most once per second for assistive tech. |
| Meter loop | `audio/meters.js` | one loop | The one shared `requestAnimationFrame` loop for all meters on the page. It also runs the reader of each mounted instrument's ring buffer of `{ stepIndex, time }` (playhead, hit flashes). `ui/vu.js` has no loop of its own. It toggles a class only when a segment changes (the sources toggle all 20 every frame). It stops when `document.hidden` and when no meter is registered. | none |
| Pad | `ui/pad.js` | square, `aspect-ratio: 1/1`, 36 px cell in the beat grid, radius `--r-pad`, 1 px `--stroke` border | A `button` in a `--recess` well with inset shadow; an `::after` fill rises from the bottom (height = value from 0 to 1 as a percentage; the source draws the half level at 48 %; colour `--c`, opacity .88). Value 1 adds glow. `.playhead` draws a 2 px `--ink` outline, offset 1 px. `.hit` sets `filter: brightness(1.9)` for 130 ms. `:active` scales .96. | Native button. Enter and Space cycle the value (0, 1, 0.5, 0); `set(value)` takes any value from 0 to 1. `aria-pressed` is `false` for 0, `true` for 1 and `mixed` otherwise. `aria-label` is "&lt;channel&gt; step &lt;n&gt;". |
| LCD | `ui/lcd.js` | height 30 px (SM `.lcd`), min-width 52 px, width set by the container (SM info LCD is 230 px, header LCD 200 px, new value) | A `div` in `--lcd` with 1 px `--black` border, inset shadow, 1 px `--stroke` ring, `--font-display` 12 px, red text with 6 px glow (`tone: "red"`) or amber text (`tone: "amber"`). | `role="status"` with `aria-live="polite"` when `live: true` (header status, octave); otherwise plain text. Text is never only a glyph. |
| Buttons | classes in `components.css` | transport 38 px square; step 24 by 22 px; toggle (mute, solo, LO-CUT) 34 px square; pill (RND, CLR) height 34 px, min-width 38 px (BM); primary (mixdown, load) padding 10 by 20 px | Gradient `--btn-top` to `--recess`, 1 px `--stroke-bright` border, radius `--r-btn`, hard 2 px bottom shadow that collapses to 1 px and moves 1 px down on `:active` (BM `.tbtn`). Toggle active: red (mute, LO-CUT) or amber (solo) background with `--on-accent` text. Disabled: opacity .35. | Native `button`. Toggles use `aria-pressed`. Glyph-only buttons carry `aria-label`. Focus ring 2 px `--accent`, offset 2 px; knob ring offset 3 px. |

### 4.5 Shared effect modules

`fx/reverb.js` and `fx/delay.js` exist once and take one parameter set. Each caller passes its own values, so every instrument keeps the effect sound of its source. The ranges of the knobs stay in the instrument specs.

| Module | Call | Behaviour |
|---|---|---|
| `fx/reverb.js` | `createReverb(ctx, { decay, preDelay, power, toneHz, mix })` | Generates a noise impulse response with the given decay time, pre-delay and decay power. A change of `decay`, `preDelay` or `power` rebuilds the response after 150 ms without further input. `toneHz` and `mix` change at once. The maximum decay is a parameter of the caller (BM 4 s, SM 6 s, MX 8 s). Console and Slicer map their own controls onto these values. |
| `fx/delay.js` | `createDelay(ctx, transport, { division, feedback, toneHz, mix })` | Tempo-synced delay. It retimes itself on a tempo change through `transport.subscribe`; instruments never retime it. The list of divisions is a parameter of the caller (BM 11, SM 10). |

## 5. State

The shell owns two things in the project: the transport settings (tempo, swing) and one on/off flag per instrument. Everything else is instrument state. The project JSON is one plain object; the shell part is the keys `schemaVersion`, `transport` and `on`. The name "song" is not used; the state is a project. `schemaVersion` is the only version key and belongs to the project. The switch is stored as `on.<id>`, not inside the state of the instrument. The Console has no switch. Which tab is shown is interface state and never enters the project. Audio files are referenced by the key `sampleId`.

```json
{
  "schemaVersion": 1,
  "name": "demo-project",
  "transport": { "bpm": 120, "swing": 0 },
  "on": { "beat": true, "synth": false, "slicer": false, "theremin": false, "metronome": false },
  "instruments": {
    "beat": { "tracks": [] },
    "synth": { "wave": "sawtooth" },
    "console": { "strips": [] }
  }
}
```

The "instruments" values shown here are placeholders; each instrument spec defines its own shape. Allowed ranges: `bpm` 40 to 300 integer; `swing` 0 to 70 integer. The `on` default for each instrument is in Open questions. Nothing about colour, zoom or knob position is stored; the knobs re-read their value from the instrument `setState`.

## 6. Audio graph

`audio/engine.js` creates the one `AudioContext` at page load and resumes it on the first user gesture anywhere on the page. The shell does not create it. `audio/mixer.js` exports `buildMixer(state, ctx)`: channel strips, send reverb, and the master bus with fader, limiter and meter taps. The engine hands each instrument its strip input as `output`. The Console is only the user interface of the mixer; it builds no strips and never connects to `ctx.destination`. No Web Audio node exists in the shared UI components, except the analysers that `vu.js` reads.

```
instrument output = strip input  (per instrument and per WAV track: gain, pan, EQ, reverb send, mute, solo)
   -> masterInput (GainNode, 1.0)
   -> masterFader (GainNode, default 1.0 = 0 dB, range silence, -60 dB to +6 dB, taper db)
   -> limiter
   -> ctx.destination

limiter -> ChannelSplitter(2) -> AnalyserNode L (fftSize 512)
                              -> AnalyserNode R (fftSize 512)
   header master meter reads L and R through audio/meters.js
```

The values owned by this specification and docs/modules/console.md are: strip default gain 0.8 (about -1.9 dB), master fader default 1.0 (0 dB), reverb return default 1.0, and the meter range -60 to +6 dB for the master meter and the Console strips. Other specifications refer to these values and state no different ones. A voice inside an instrument connects to that instrument's `output`, as described in the instrument specs. The BM fixed master gain of 0.9 stays an internal Beat bus gain, and the SM -4 dB stays a fixed trim of gain 0.631 inside the synth chain; neither is a strip default. The fftSize 512 is the MX master value; BM uses 256 for its RMS meter (5 ms of audio per 16 ms frame at 48 kHz, which MX names as a miss risk). `smoothingTimeConstant` is not set (it has no effect on `getByteTimeDomainData`, BM l.899 and 902).

## 7. Transport behaviour

The lookahead scheduler itself is described in docs/plan.md, section Transport (tempo 40 to 300 BPM, default 120; swing 0 to 70 %, default 0; tick 25 ms in a Web Worker; lookahead 0.2 s; `stepIndex` counts up since start). This section says what the shell does around it.

On Start click, the shell calls `transport.start()`. The engine has already resumed the context on the first user gesture on the page, so the shell does not resume it. The Start button becomes disabled, Stop becomes enabled, the run LED turns on and the status LCD shows RUNNING. There is no countdown and no wait for a wall-clock boundary. The first step (index 0) plays at the audio time of the click plus a short lookahead set by the transport module.

On Stop click, the shell calls `transport.stop()`, which sets the stopped state first and then calls `onStop()` on every switched-on instrument. After that no further `onStep` call is made, and no step callback may change the status LCD or the run LED; only the shell writes them, from the Start and Stop clicks. The status LCD shows STOPPED, the run LED turns off, Stop becomes disabled and Start becomes enabled. The shell cancels no audio by itself; each instrument cancels its scheduled voices and ends its sounding voices with a short fade in `onStop()` and clears its ring buffer (see Defects that broke behaviour). Reverb and delay tails may ring out. The Theremin is the exception: the transport does not silence it, only its own switch does.

On each step, the transport calls `onStep(stepIndex, time, stepDuration)` on every instrument whose switch is on. `onStep` does audio only. The shell does not draw the playhead; a mounted instrument interface reads the instrument's ring buffer of `{ stepIndex, time }` in the shared animation loop of `audio/meters.js`. An instrument whose tab is not shown keeps playing because audio code does not depend on mounted DOM.

On a switch change, the shell writes `on.<id>` into the project. Switching off calls `onStop()`. Switching on while the transport runs calls `onStart(time)`, and the instrument plays from the next `onStep` with the current `stepIndex`.

On tempo or swing change, the header knob or button writes the value into the transport, which applies it from the next step. `fx/delay.js` retimes itself through `transport.subscribe`. The shell updates the knob `aria-valuetext` and the `+` and `-` button limits (disabled at 40 and 300).

On tab switch, the shell calls `unmount()` on the old instrument and `mount(el)` on the new one. The old instrument keeps its audio and its on/off state. The tab LED shows the on/off state of the instrument, not the shown tab.

While the browser tab is hidden (`document.hidden`), the shell stops the shared meter loop and any playhead loop, and starts them again on `visibilitychange`. A throttled `setInterval` can starve a lookahead scheduler of only 120 ms (BM l.1697, 730); docs/plan.md, section Transport, solves this with the Worker timer and the 0.2 s lookahead, and puts the one late-step guard (skip a step more than 5 ms late) in the transport. No instrument has a guard.

## 8. Defects fixed

Line references are the report line numbers. Rows whose fix is in another module say so. Only fixes of defects that broke behaviour go to `CHANGELOG.md`, with the file and line of the original. The clean-ups of the rewrite are listed in section 8.2 and not in the changelog.

### 8.1 Defects that broke behaviour

| Defect | Source and line | Fix |
|---|---|---|
| Knob wheel handler always calls preventDefault and blocks page scroll | BM l.1283-1286, SM l.703-706 | Wheel works only while the knob has focus. |
| Right-click starts a knob drag, no pointer capture | BM l.1260 | Pointer events, `setPointerCapture`, primary button only. |
| No `touchcancel`, drag state stays after a cancelled touch; mixed mouse, touch and pointer models | SM l.693-694 | Pointer events cover cancel; `pointercancel` ends the drag. |
| Stop does not stop: scheduled notes up to 120 ms ahead still sound, the visual queue is not cleared, a late step callback can set the playhead and the status back to RUNNING after Stop; SM voices ring for up to about 7 s | BM l.1657-1663, 1700-1713; SM l.1213-1220, 619 | `transport.stop()` sets the stopped state before it calls `onStop()`, and `onStep` is never called afterwards. Only the shell writes the status LCD. Each instrument cancels its scheduled voices and ends its sounding voices with a short fade in `onStop()` (not a click) and clears its ring buffer; reverb and delay tails may ring out. The Theremin is not silenced by the transport, only by its own switch. |
| setInterval scheduler with 120 ms lookahead, throttled in hidden tabs; no late-step guard, so a stall fires a burst of past steps; visual queue grows without bound while hidden | BM l.730, 1082-1087, 1697; SM l.469, 1183, 1211 | Solved in docs/plan.md, section Transport: Worker timer (25 ms tick), lookahead 0.2 s, and one late-step guard in the transport (skip a step more than 5 ms late). Instruments have no guard. The shell stops the visual loops when hidden. |
| Samples cut at decay + 0.05 s; file input not reset, so choosing the same file again does nothing | BM l.1031, 1421-1434 | Fix in the beat spec (reset the input value after load; the cut at the DECAY range is decided there). |
| Loop playback rebuilt from the main thread with a 60 ms gap, double restart race, seek without fade clicks, STOP loses the position | MX l.566, 574, 594, 599-610, 641-647 | Fix in the Console spec (follow the global transport, `source.loop` or the scheduler, short fade on seek). |
| Offline mixdown hard-clips without warning, no progress, minimal error handling | MX l.1163-1311 | Fix in the Console spec. |

### 8.2 Rewrite and accessibility changes

| Change | Source and line | Fix |
|---|---|---|
| Third-party Google Fonts request, leaks the IP and breaks offline use | BM l.2, SM l.7, MX l.6 | Fonts served from `fonts/` with `@font-face`; the page makes no third-party request. |
| No doctype, html, head, body, charset, viewport (quirks mode, mojibake risk) | BM l.1, l.1301, l.1412; SM l.311-313, l.1240 | `index.html` has `<!doctype html>`, `<html lang="en">`, charset UTF-8 and a viewport meta. |
| Knob listeners added to `window` per knob and never removed (about 260 listeners with 32 strips) | MX l.752-757 | Capture is on the knob element; the listener exists only during a drag; `destroy()` removes the rest. |
| Knobs are plain divs with no role, tabindex, aria-value or keyboard | MX l.710-761 | `ui/knob.js` role, tabindex, keyboard, aria-valuetext (also fixes missing aria-valuetext, BM and SM l.669-720). |
| Rotated native range inputs have no label and a rotated hit area | MX l.183-190 | Vertical fader uses `writing-mode`, has an `aria-label` and a normal hit area. |
| Fader geometry constants must be kept in sync with CSS by hand | MX l.374-385 | The dB scale positions are computed from the taper and the CSS custom properties `--fader-h` and `--cap-h`. |
| One rAF loop per strip plus master plus timeline; loops never cancelled; run while tab is hidden; all segments toggled every frame; a gradient allocated per frame | MX l.915-922, 1031, 421; BM l.783-793, 1716-1730; SM l.653-654 | One shared loop in `audio/meters.js` that toggles only changed segments and stops when hidden or with no meter. DOM segments need no canvas gradient. |
| Colour literals in JS and CSS drift apart (`ACC`, hard-coded hex, rgba reds) | BM l.1492-1495; SM l.473, 820, 939, 948; MX l.422 | All colours in `tokens.css`; JS reads tokens. |
| Status text and countdown not live regions | BM l.581-582; SM `#syncStatus`, `#syncCountdown` | Header status LCD has `role="status"`, `aria-live="polite"`. |
| 'LOCKED @' shows a planned boundary, not a measured start; no re-sync, anchor read while the context is suspended, UI stuck on LOCKING in a hidden tab | SM l.1192-1195, 1201-1204; BM l.1673-1681 | Removed with the wall-clock start and the countdown. |
| Reverb impulse response rebuilt synchronously on every knob tick, random and non-deterministic | BM l.1509, 1513, 848-852; SM l.834-836, 545-555; MX l.459 | `fx/reverb.js` rebuilds debounced by 150 ms (MX used 120 ms). The random noise stays (see Kept on purpose). |
| No error handling for a missing or blocked Web Audio, `resume()` ignored, `setPointerCapture` unguarded | SM l.560, 598, 1044, 1119; BM (no feature check) | `audio/engine.js` creates the context in a try/catch and resumes it on the first user gesture; the shell shows "Web Audio not available" on the status LCD when the engine reports a failure. `ui/knob.js` guards `setPointerCapture`. |
| Loaded audio is lost on reload, no save or load, global mutable IIFE state that cannot be serialised | BM l.724-1732; SM l.465-1237; MX l.480-539 | Solved by the target structure: one `createInstrument` per instrument, `getState` and `setState`, and the project backend (audio files by `sampleId`). |
| Duplicated code (hp and lp cut mapping, reverb and delay builders, 19 makeKnob calls, live and offline graph) | BM l.832-833, 1501-1571; SM l.737-859; MX l.482-503, 1199-1237 | Shared modules `ui/*`, `fx/*`; `buildMixer` serves live and offline. |
| Note creation, selection and velocity lane only by pointer | SM l.1034-1038, 1083-1094, 1112 | Fix in the synth spec (keyboard access). |
| Voice nodes never disconnected | BM l.1049; SM (voice gains) | Fix in the instrument specs (disconnect on `ended`). |
| Low contrast: `--ink-faint` on plate about 3.5:1; strip number #000 on metal | BM, SM, MX text of 9 to 11 px | `--ink-faint` only on `--recess` and floor; labels on plates use `--ink-dim`; strip numbers use `--ink`. |
| `user-select: none` on the whole page, including file names | MX l.24 | `user-select: none` only on controls; names and readouts stay selectable. |
| Fixed 1180 px stage, no responsive layout, horizontal page scroll, touch targets of 20 to 26 px | BM l.40; MX whole file | Fluid layout from 640 px up and 44 px coarse-pointer hit areas (see Layout). |
| Dutch and English strings mixed, no i18n | MX l.565, 607, 795, 798, 950, 1071, 1108 | All shell and Console strings are English (interface language decision). |
| AudioContext created at page load before any gesture | MX l.386 | `audio/engine.js` creates the one context at page load, in a try/catch, and resumes it on the first user gesture anywhere on the page. The shell does not create it. |
| Mute and solo state shown by colour only | MX l.906-912 | `aria-pressed` on every toggle. |
| Stale aria-labels after a channel rename | BM l.1314, 1322, 1332, 1342 | Labels are built with `aria-labelledby` pointing at the name element. |
| Dead UI: FX LEDs never light up | BM l.522, 1503, 1533 | `ui/led.js` has a real `on` state; instruments set it while the effect has a non-zero mix or send. |

## 9. Kept on purpose

The dark chassis with the stripe floor, the red radial glow at the top (rgba(217,46,58,.10), now `color-mix`), the red hazard strip, four rivets and the rounded 16 px unit stay, as in BM and SM. The 0.8 s power-on flicker stays (BM l.51-61, SM l.29), and `prefers-reduced-motion` turns it off (BM l.27-29).

The knob sweep of -135 deg to +135 deg, the 3 px red pointer, the 52 px size and the specular highlight stay. The meter feel stays: RMS level, 10 segments, instant attack and release, no ballistics; the peak-hold number never decays until clicked (MX). The 150 px drag distance of BM and SM stays instead of the 140 px of MX. The dB fader taper with 0.75 at 0 dB stays (MX). The cream fader cap of the console stays; parameter faders keep the red round thumb. Michroma for display text and Barlow for body text stay, with the uppercase 10 px label with letter-spacing 1px. The 0.5 s `steps(1)` LED blink stays. The mute colour red and the solo colour amber stay.

Sound quirks of the sources stay and are not defects of this module. The filter resonance maps to Q 0.7 to 18 although Web Audio reads Q in dB (BM l.834, SM l.495). Reverb impulse responses stay random noise with a power-law or exponential decay, so each rebuild sounds slightly different (BM, SM, MX l.459). The SM synth keeps one oscillator per voice, the polyphony cap of 16 with silent dropping of extra notes, stacking of overlapping notes of the same pitch, and the compressor as its only guard. The master bus has a limiter, built by `audio/mixer.js`. The visual playhead is not compensated for output latency, as in the sources. The mute, solo and sample changes of the beat take effect up to the lookahead time late.

## 10. Layout

### 10.1 Shell sketch

Desktop at 1180 px width and above. The header is fixed at the top (height 88 px, new value to tune). The chassis hazard strip (4 px) sits above the header.

```
+------------------------------------------------------------------------------+
|==== hazard strip (4 px, red gradient) =======================================|
| (o)                                                                      (o) |
|  [logo]  RABBIT STUDIO     [>] [#]  (TEMPO) [-][+]  (SWING)  +---------+ ||||| |
|          six instruments,  Start Stop  120 BPM        0 %    |STOPPED  | ||||| |
|          one page         (*)                                 +---------+ L R  |
|--------------------------------------------------------------------------------|
|  BEAT (*)   SYNTH (*)   SLICER ( )   THEREMIN ( )   METRONOME ( )   CONSOLE (*) |
|--------------------------------------------------------------------------------|
|                                                                                |
|   content area: the active instrument mounts here (mount(el))                  |
|                                                                                |
|   +-------------------------------+  +-------------------------------------+   |
|   | PLATE  (rack, left)           |  | PLATE  (rack, right)                |   |
|   | (K) (K) (K) (K)               |  | (K) (K) (K) (K)                     |   |
|   +-------------------------------+  +-------------------------------------+   |
|   +-----------------------------------------------------------------------+    |
|   | full-width plate, for example the 16 step pad rows                    |    |
|   +-----------------------------------------------------------------------+    |
|                                                                                |
| (o)    legend line in amber                                                (o) |
+------------------------------------------------------------------------------+
```

Symbols: `(*)` lit LED, `( )` unlit LED, `(K)` knob, `[>]` start, `[#]` stop, `|||||` master meter (two rows of 10 segments).

A plate (rack unit) in the RABBIT look: a vertical rotated name rail with an LED, a 1 px divider, then wrapped controls.

```
+--+--+----------------------------------------------+
|V |  | (K)   (K)   (K)   [M][S]   ===o===  ==o==     |
|N |  | VOL   PAN   ...   mute  solo  fader   fader   |
|A |  |                                                |
|M |  | [pad][pad][pad][pad] [pad][pad][pad][pad] ...  |
|E |  |                                                |
+--+--+----------------------------------------------+
```

### 10.2 Layout rules

The page is `max-width: 1180px`, centred (fluid below that, not fixed as in BM l.40). The content area is a CSS grid with rows of one or two columns. Plates use `--plate-1` to `--plate-6` and a corner screw radial gradient (BM l.326-349). The header uses `position: sticky; top: 0` inside the chassis so it stays visible; its height is a CSS custom property `--header-h`.

### 10.3 Accessibility rules

Every control is reachable with Tab in DOM order: header controls first, then the tab row, then the content. The tab row follows the ARIA tablist pattern: `role="tablist"`, `role="tab"` per tab, `aria-selected`, `aria-controls` pointing at the content area, Left and Right arrows move between tabs, Home and End go to the first and last tab. The on/off switch of each instrument is a sibling `button role="switch"` with `aria-checked` and an `aria-label` such as "Beat on", not nested in the tab button.

Focus is always visible: `:focus-visible` gives a 2 px `--accent` outline (BM l.294-296). State is never shown by colour alone: toggles use `aria-pressed`, LEDs sit beside a text label or switch. Text contrast follows the token rule (`--ink-faint` only on `--recess` and floor). Reduced motion removes the flicker, the blink and transitions. The header status LCD is a polite live region. Pad count (80 pads on the beat) uses native buttons; roving tabindex is an open question.

### 10.4 Responsive rules

The 1000 px breakpoint comes from SM (l.299). At 1000 px and below, two-column racks become one column. The smallest supported width is 640 px. There are no rules and no tests for narrower screens. The tab row becomes a horizontal scroller inside its own container when it does not fit (the page itself never scrolls horizontally). The Console strip rack (136 px per strip, MX) scrolls horizontally inside its own container, not the page. On `(pointer: coarse)` the knob, fader and button hit areas grow to at least 44 by 44 CSS px using an invisible `::before`; the visible size does not change.

## 11. Tests

All tests are pytest files in `02_backend/tests/`, run with `uv run pytest`. Browser tests use Playwright for Python. Module-level JavaScript checks import the real module inside the page and call it with `page.evaluate`. Live checks open the page with `?test=1` and read `window.__rabbit` and its `onStep` log. There is no Vitest, no `node --test` and no npm.

| Test | What it checks |
|---|---|
| `test_shell_tokens.py` | `tokens.css` defines every token of the tokens table; no colour value in any notation (hex, `rgb()`, `rgba()`, `hsl()`) exists in any `.css` or `.js` file outside `tokens.css`. |
| `test_shell_fonts.py` | `index.html` and all CSS contain no `googleapis` or `gstatic` URL; every `@font-face` file exists in `fonts/`. |
| `test_knob_keyboard.py` | Arrow keys change one step, Shift changes five, Home and End hit min and max, values clamp. |
| `test_knob_aria.py` | Role, tabindex, `aria-valuemin`, `aria-valuemax`, `aria-valuenow` and `aria-valuetext` are set and update after each change. |
| `test_knob_reset.py` | Double-click sets `reset`; with `reset: null` it changes nothing; with `reset` unset it sets `default`. |
| `test_knob_pointer.py` | Drag of 150 px covers the full range; a non-primary button does not start a drag; `pointercancel` ends a drag; no listener remains on `window` after the drag. |
| `test_knob_wheel.py` | Wheel changes the value only with focus; without focus `preventDefault` is not called. |
| `test_fader_taper.py` | For `taper: "db"`: position 0.75 gives gain 1.0, position 1.0 gives 1.995 (+6 dB), position 0.001 or less gives 0; `gainToPos(posToGain(p))` equals p within 0.001. |
| `test_fader_aria.py` | The vertical and the compact vertical fader have a name and `aria-orientation="vertical"`; the channel fader has `aria-valuetext` in dB; the horizontal fader value text follows `format`. |
| `test_led.py` | States off, on and blink set the right classes and `aria-hidden="true"`. |
| `test_vu.py` | With a fake analyser and the range -60 to +6 dB: silence lights 0 segments, a level at or above `max` lights 10, mid values light round(level * 10); the Slicer range -60 to 0 lights 10 at 0 dB; segment colours by index follow 1 to 6, 7 to 8, 9 to 10; a meter registers on mount, deregisters on unmount and has no loop of its own. |
| `test_meters.py` | One rAF loop exists for any number of meters; it stops when `document.hidden` is true and when the last meter is removed; a segment class is toggled only when its state changes. |
| `test_pad.py` | Clicks cycle 0, 1, 0.5, 0; `aria-pressed` is false, true, mixed in the order 1 then 0.5; fill height follows the value; `playhead` and `hit` classes are set and `hit` is removed after 130 ms. |
| `test_lcd.py` | `set()` updates text; `live: true` gives `role="status"`. |
| `test_shell_tabs.py` | The six tabs exist; Left and Right arrows move the tab focus; switching a tab does not stop a switched-on instrument (the `window.__rabbit` `onStep` log keeps growing); switching an instrument off calls `onStop`, and switching it on while running calls `onStart`; the header stays visible after scroll. |
| `test_shell_transport.py` | Start enables Stop and disables Start; the status LCD reads RUNNING then STOPPED; after Stop the instrument gets `onStop()`, the `onStep` log gets no new entry, and the LCD stays STOPPED; tempo buttons clamp at 40 and 300; Shift plus button changes by 5. |
| `test_shell_responsive.py` | At 640, 1000 and 1280 px width, `document.documentElement.scrollWidth` is not greater than the viewport width. |
| `test_shell_a11y.py` | An automated accessibility scan reports no serious violation on the shell with the beat tab shown; contrast of `--ink-dim` on `--plate-1` is at least 4.5:1. |
| `test_shell_offline.py` | With all network requests except localhost blocked, the page loads with Michroma and Barlow (checked with `document.fonts.check`). |

## 12. Open questions

1. Font files. The sources give no font files, only the Google Fonts link. Michroma and Barlow need woff2 files (both are open licence fonts) placed under `01_frontend/fonts/`. Who downloads them once and commits them? Exact file sizes and subsets (Latin only?) are TBD.
2. Logo. BM (about 20 KB, l.572), SM (about 20 KB, l.321) and MX (about 12 KB, l.327) each embed a base64 PNG. It is not clear if they are the same image. One `img/logo.png` is assumed; TBD which one.
3. Default on/off state of each of the five switchable instruments at page load. No source says it. Proposal: Beat on, the others off. TBD.
4. Header dimensions (88 px height, 12 by 8 px horizontal segments, 200 px LCD width) are new values of this spec, not from a source. They need a check in the browser.
5. Wheel on knobs. The sources always capture the wheel; this module only does so with focus (to keep page scroll). Is that change acceptable?
6. Minimum label size. Sources use 9 to 12 px; this spec sets 10 px. WCAG does not set a minimum size; should it be 11 px?
7. Pad focus. 80 pads in the beat grid are 80 Tab stops. Use roving tabindex with arrow keys, as in a grid? No source has it.
8. Knob extra keys. Page Up and Page Down are not in any source; add them for the slider pattern? Without them a sweep of the tempo range needs about 260 key presses (BM report).
9. Fader implementation. The vertical fader uses `writing-mode: vertical-lr` on a native range input. This needs a check in the target Chromium and Firefox versions; the fallback is a `role="slider"` div.
10. VU thresholds. The sources light round(level * 10) segments with no dB mapping; the peak readout range -60 to +6 dB is from MX. No source gives dB thresholds per segment, so the segments map linearly between the dB range that the caller passes. Is that acceptable?
11. Accessibility scan library. The a11y test needs a scan library for Python (for example a vendored axe-core script run by Playwright); the rule is not to add dependencies unasked. Which one is approved?
