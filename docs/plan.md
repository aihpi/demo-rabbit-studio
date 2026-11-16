# Build plan for RABBIT Studio

This plan describes how to rebuild the eight browser instruments of Tom Tiest as one application in one Docker container. It is written before any application code exists. Read it together with [inventory.md](inventory.md), which lists what the original files do, the module specifications in [modules/](modules/), which list what each part of the new application must do, and [open-questions.md](open-questions.md), which lists what is still undecided.

This plan is the single source for decisions, names of files and functions, and the order of work. Where a module specification contradicts this plan, the plan wins and the specification is corrected.

New to web development? Read Goal, The two approaches side by side, Scope and Stages. The section Architecture is for the person who builds a stage; the README explains its words.

## Goal

Tom Tiest is a musician and teacher. He built each instrument as one HTML file with an AI chatbot, and he keeps several of them in time with a trick: every instrument starts only when the seconds of the computer clock are a multiple of four. This works for a first prototype. It does not survive a tempo change, it cannot save anything, and the same building blocks exist several times in different versions.

RABBIT Studio keeps everything the eight files can do and keeps their look. It replaces the trick with one shared clock. The repository is a teaching example for the workshop series Rapid Prototyping via Vibe Coding: it shows which decisions a chatbot does not take for you.

## The two approaches side by side

| Question | Eight HTML files | RABBIT Studio |
|---|---|---|
| How do instruments stay in time? | Each file has its own clock. Only the start is aligned to the wall clock. | One transport drives every instrument. |
| What happens on a tempo change? | The files drift apart. | Every instrument follows at once. |
| Where is the reverb? | Written four times, each version different. | One module, used by every instrument. |
| Can I save my pattern? | No. A reload loses everything. | Yes, as a named project. |
| Can the mixer hear the instruments? | No. Each file is a separate page. | Yes. Each instrument has a channel strip. |
| How do I start it? | Open eight files. | `docker compose up`, then one address. |
| Who can change it? | Whoever understands one file of up to 1,733 lines. | Anyone who reads one module of the plan and one folder. |

## Decisions

These decisions were taken with Hanno Müller on 2026-11-16. A change to one of them is a change to this plan and needs a new decision.

| Topic | Decision | Reason |
|---|---|---|
| Shape | One page with one tab per instrument inside the application. | All instruments share one audio engine, so they are in time by construction. |
| Synchronisation across computers | Not part of the application, and not planned. | A network clock is a separate problem. |
| Wall-clock start | Removed, together with its countdown. | One transport replaces it. The original files stay in `03_originals/` for comparison. |
| Frontend | Plain HTML, CSS and JavaScript modules. No build step, no framework, no TypeScript, no Node tooling. | Tom's code is plain JavaScript on the Web Audio API. He and other beginners can read it. |
| Backend | FastAPI in Python, managed with `uv`. SQLite and a folder for files. | The smallest backend that can save projects and samples. |
| Container | One container, started with `docker compose up`. | One thing to start, one address to open. |
| Modules | Six instruments: Beat, Synth, Slicer, Theremin, Metronome, Console. | The two generations of files overlap. Eight modules would copy the duplication. |
| Look | The RABBIT chassis of the three newest files for every instrument. | One design system. The five older files lose their neon colours and keep their functions. |
| Playing in the background | An instrument that is switched on keeps playing while another tab is shown. | This is what separate pages cannot do reliably. |
| Console | One channel strip per instrument next to the WAV tracks. The WAV mixdown includes the instruments. | The clearest gain of one application. |
| Sound | Tom's sound code keeps its character. Only defects that break behaviour are fixed, and each fix is listed in `CHANGELOG.md`. | The sound is his work. Correctness is ours. |
| Language and fonts | English interface. Michroma and Barlow are served from the container. | No request to Google when the page loads. |
| Saving | A project holds the state of all six instruments. It is saved and loaded by name. | No original file saves anything. |

## Scope

The application must offer every function of the eight original files. The feature matrix in [inventory.md](inventory.md) is the checklist, and each module specification has a parity table that maps every source feature to its place in the application. A source function that a specification leaves out is an exception that the project owner must accept; the pending ones are in [open-questions.md](open-questions.md).

| Module | Comes from | Specification |
|---|---|---|
| Beat | `RABBIT-BeatMachine_x.html`, `CP5_TT_20251118_DrumSEQ4.html` | [modules/beat.md](modules/beat.md) |
| Synth | `RABBIT-SynthMachine1_Full.html`, `CP5_TT_20251118_SynthSEQ3.html` | [modules/synth.md](modules/synth.md) |
| Slicer | `CP5_TT_20251118_Slicer5_update20260524.html` | [modules/slicer.md](modules/slicer.md) |
| Theremin | `CP5_FD_20251104_theremin01.html` | [modules/theremin.md](modules/theremin.md) |
| Metronome | `CP5_TT_20251119_Metronoom5.html` | [modules/metronome.md](modules/metronome.md) |
| Console | `RBBT-MixConsole1_Full.html` | [modules/console.md](modules/console.md) |
| Shell and design system | the three RABBIT files | [modules/shell.md](modules/shell.md) |

Out of scope: user accounts, several users on one project, synchronisation over the network, layouts for screens narrower than 640 px, MIDI, and audio recording from the microphone.

## Architecture

### One picture

```
Browser (one page)                                  Container
┌─────────────────────────────────────────────┐    ┌──────────────────────────┐
│ Shell: header with transport, tab row        │    │ FastAPI                  │
│ ┌────────┐ ┌───────┐ ┌────────┐ ┌─────────┐ │    │  /            frontend   │
│ │ Beat   │ │ Synth │ │ Slicer │ │ ...     │ │    │  /api/projects  SQLite   │
│ └───┬────┘ └───┬───┘ └───┬────┘ └────┬────┘ │◄──►│  /api/samples   files    │
│     │ onStep   │         │           │      │    │                          │
│ ┌───┴──────────┴─────────┴───────────┴────┐ │    │ volume: data/            │
│ │ Transport: one clock, tempo, swing       │ │    └──────────────────────────┘
│ └──────────────────────────────────────────┘ │
│ ┌──────────────────────────────────────────┐ │
│ │ Engine: one AudioContext, the mixer       │ │
│ │ instrument → channel strip → master → out │ │
│ └──────────────────────────────────────────┘ │
│ Store: one project object, saved as JSON     │
└─────────────────────────────────────────────┘
```

### Folder layout

```
demo-rabbit-studio/
├── 00_aisc/                 logos of the AI Service Centre
├── 01_frontend/             everything the browser loads; plain files, no build
│   ├── index.html
│   ├── app.js               starts the engine and mounts the shell
│   ├── styles/              tokens.css, chassis.css, components.css
│   ├── fonts/  img/         Michroma, Barlow, img/logo.png
│   ├── core/                store.js (project state), api.js (calls to the backend)
│   ├── audio/               engine.js, mixer.js, transport.js, clock-worker.js,
│   │                        meters.js, noise.js, wav.js
│   ├── fx/                  reverb.js, delay.js
│   ├── ui/                  knob.js, fader.js, led.js, vu.js, pad.js, lcd.js,
│   │                        shell.js, header.js, tabs.js
│   └── instruments/         beat/ synth/ slicer/ theremin/ metronome/ console/
├── 02_backend/              the Python server
│   ├── pyproject.toml  uv.lock
│   ├── Dockerfile
│   ├── app/                 main.py, db.py, projects.py, samples.py
│   └── tests/               backend tests and browser tests, all run by pytest
├── 03_originals/            Tom Tiest's eight files and his prompts, unchanged
├── docs/                    this plan, the inventory, the module specifications
├── .dockerignore
├── docker-compose.yml
└── README.md
```

Each instrument folder holds `index.js` and further JavaScript files of that instrument. Styles live only in `styles/`. With no bundler, these paths are what the import statements use, so a file that is not in this tree does not exist.

The template ships a React Dockerfile in `01_frontend/`, a GPU Dockerfile and `requirements.txt` in `02_backend/`, and a two-service `docker-compose.yml`. Stage 0 replaces all four.

### Audio engine and mixer

`audio/engine.js` creates the only `AudioContext` when the page loads. Browsers keep a new context silent until the user acts, so the engine resumes it on the first click or key press anywhere on the page, not only on start. This matters for the Theremin, which makes sound without the transport.

`audio/mixer.js` exports `buildMixer(state, ctx)`. It builds one channel strip per instrument and per WAV track (gain, pan, equaliser, reverb send, mute, solo), the send reverb, and the master bus with fader, limiter and meter taps. The engine calls it before any instrument exists and hands each instrument the input of its strip as `output`. The Console module is the user interface of this mixer, plus the WAV tracks, their timeline and the mixdown. It builds no strips of its own and never connects to `ctx.destination`. The mixdown calls the same `buildMixer` with an `OfflineAudioContext`.

`audio/meters.js` runs the one animation loop for all meters. A meter from `ui/vu.js` registers when its tab is shown and deregisters when it is hidden. The mixer owns the meter taps of the strips and the master. An instrument may keep a meter tap and dynamics of its own inside its chain where its source file had them, for example the compressor of the Synth. The camera analysis of the Theremin is the one further loop in the application, because it must run while the tab is hidden.

### Transport

The transport owns time. Tempo runs from 40 to 300 BPM with a default of 120, and swing from 0 to 70 % with a default of 0. These are the widest ranges of the original files. No instrument has a tempo or swing control.

It runs a lookahead scheduler: a timer wakes up every 25 ms and schedules all steps of the next 0.2 s on `audioContext.currentTime`. The RABBIT files use the same technique with 0.12 s. The timer runs in a Web Worker (`audio/clock-worker.js`), because browsers slow down the timers of a hidden page to about one tick per second. For each sixteenth-note step the transport calls `onStep(stepIndex, time, stepDuration)` on every instrument that is switched on. `stepIndex` counts up from 0 since start and is never reset by an instrument; `time` already contains the swing.

A step whose time is more than 5 ms in the past when the transport gets to it is skipped, and the counter moves on. This guard exists once, in the transport. Instruments have no guard of their own.

The transport offers instruments a read-only surface: `bpm`, `swing`, `running`, `stepDuration()` and `subscribe(callback)` for tempo and swing changes. `fx/delay.js` uses `subscribe` to retime a tempo-synced delay; instruments do not retime it themselves.

`createOfflineTransport({ bpm, swing })` in the same file returns the same surface without a timer. The mixdown uses it.

### The instrument contract

Each instrument is one folder. Its `index.js` exports one function:

```js
export function createInstrument({ ctx, output, transport, store }) {
  return {
    id: "beat",
    title: "Beat",
    ready() {},             // a promise; resolves when samples are decoded
    mount(element) {},      // build the user interface inside element
    unmount() {},           // remove it; the audio keeps running
    onStart(time) {},       // audio time of the first step
    onStop() {},            // silence every voice that is still sounding
    onStep(stepIndex, time, stepDuration) {},   // audio only
    getState() {},          // a plain object that JSON can store
    setState(state) {},
  };
}
```

`ctx` may be the live context or an offline one, so an instrument never keeps a context of its own and never reads the clock of the computer.

`onStep` schedules audio and nothing else. For the playhead and for flashing pads, the instrument writes `{ stepIndex, time }` into a small ring buffer. A mounted interface reads the buffer in the shared animation loop and drops entries older than one second. An instrument that is not mounted, or that runs offline, has no reader and does no interface work.

The switch of an instrument works like this: switching off calls `onStop`. Switching on while the transport runs calls `onStart(time)`, and the instrument plays from the next `onStep` with the current `stepIndex`. The Console has no switch; it is always on.

### Mixdown

The WAV export builds the song a second time:

1. Create an `OfflineAudioContext` with the sample rate of the live context.
2. Call `buildMixer(state, offlineCtx)`.
3. Call `createInstrument` for every instrument that is switched on, except the Theremin, with the offline context, its strip and an offline transport. Call `setState` and wait for `ready()`.
4. Call `onStart(0)`, then `onStep` for every step of the range.
5. Render, encode with `audio/wav.js`, and offer the file.

The Theremin depends on a live camera and is left out; the status line says so. The Slicer picks slices at random, so its part of the mixdown has the same settings as what was heard, but not the same slices.

### Shared building blocks

These exist once. The values, sizes and behaviours are in [modules/shell.md](modules/shell.md).

| Module | Interface in outline |
|---|---|
| `fx/reverb.js` | `createReverb(ctx, { decay, preDelay, power, toneHz, mix })`. It generates a noise impulse response. Each caller passes its own values, so every instrument keeps the reverb sound of its source. A change of `decay`, `preDelay` or `power` rebuilds the response after 150 ms without further input. |
| `fx/delay.js` | `createDelay(ctx, transport, { division, feedback, toneHz, mix })`. It retimes itself on a tempo change. |
| `audio/noise.js` | Noise buffers for drums and reverb. |
| `ui/knob.js` | Rotary control with keyboard and ARIA support. `reset: null` means a double-click does nothing; without the option a double-click restores the default. |
| `ui/fader.js` | Three sizes: horizontal parameter fader, vertical channel fader, compact vertical fader for step velocity. |
| `ui/pad.js` | Step cell with a value from 0 to 1, shown as fill height. A click cycles 0, 1, 0.5. |
| `ui/vu.js` | Meter with 10 segments. The caller passes the range in dB. |
| `ui/led.js`, `ui/lcd.js` | Status light and text display. |
| `styles/components.css` | Buttons, toggles and selectors are plain HTML buttons with shared classes. `ui/pad.js` is only for step cells. |

### State

One store holds the project: a plain object with `schemaVersion`, the transport settings, the switch of each instrument (`on.beat`, `on.synth` and so on) and one entry per instrument. `schemaVersion` is the only version key. Everything the user would expect to find again after a reload is in the project. Everything else (which tab is open, which view of an instrument is shown, the position of the playhead, a knob that is being dragged) is interface state and never enters the project.

An audio file is referenced by the key `sampleId`, which is the id the backend returned on upload.

### Backend

The backend has two jobs: it hands the frontend files to the browser, and it stores projects and audio files.

| Method and path | Purpose |
|---|---|
| `GET /` | the frontend |
| `GET /api/health` | answers `{"status": "ok"}`; used by tests and by Docker |
| `GET /api/projects` | list of saved projects (id, name, time of last change) |
| `POST /api/projects` | save a new project |
| `GET /api/projects/{id}` | load one project |
| `PUT /api/projects/{id}` | overwrite one project |
| `DELETE /api/projects/{id}` | delete one project |
| `POST /api/samples` | upload one audio file; the answer contains its `sampleId` |
| `GET /api/samples/{id}` | download one audio file |

Projects go into one SQLite file, audio files into one folder. Both live in the Docker volume `data`, so they survive a restart of the container.

The backend checks what it receives. A project must be valid JSON with a known `schemaVersion` and at most 1 MB. Every `sampleId` in it must exist; otherwise the answer is status 422. An upload must be an audio type and at most 50 MB. When a project is saved or deleted, the backend removes audio files that are older than one day and that no project references.

The frontend files are sent with `Cache-Control: no-cache`, so the browser never mixes old and new modules after a change. The backend registers the file types `.js`, `.css` and `.woff2` at start, because some systems report them wrongly. The route for the frontend is added last, after all `/api` routes.

### Docker

One `Dockerfile` in `02_backend/` builds one image from a slim Python base with `uv`. The build context is the repository root, so the image can copy `01_frontend/` next to the backend code. A `.dockerignore` at the root keeps `.git`, `03_originals/`, `docs/` and virtual environments out of the build.

`docker-compose.yml` defines one service on port 8080 with one worker process, the volume `data` for `/data`, and a health check on `/api/health`. It also mounts `01_frontend/` into the container, so a saved frontend file shows up after a reload of the page without a rebuild. Two environment variables with defaults tell the backend where things are: `FRONTEND_DIR` and `DATA_DIR`.

The Theremin needs the camera. Browsers allow the camera only on `http://localhost`, `http://127.0.0.1` or an HTTPS address. A person who opens the studio from another computer by its network address gets no camera.

### Rules that keep the structure

1. There is one `AudioContext`. It is created in `audio/engine.js` and passed in. No other file calls `new AudioContext()`.
2. There is one clock. No instrument has its own tempo, its own timer for steps, or a reference to `Date.now()` for timing.
3. An instrument reaches the speakers only through the `output` it was given.
4. Audio code and interface code are separate. Audio code works without a mounted interface, and `onStep` does no interface work.
5. Reverb, delay, noise, knob, fader, LED, meter, pad and display exist once, in `fx/`, `audio/` and `ui/`. An instrument imports them and does not copy them.
6. Colours and fonts come from `styles/tokens.css`. No colour value in any notation (hex, `rgb()`, `hsl()`) appears in any other file.
7. Project state is plain data in the store. Audio nodes and page elements never enter it.
8. Stop means silence. `onStop` cancels everything that was scheduled and ends every sounding voice with a short fade; reverb and delay tails may ring out. The Theremin is the one exception: it is played live, so the transport does not silence it. Its own switch does.
9. The backend validates every request body. The frontend is not trusted.
10. Each fix of a defect that broke behaviour in an original file gets one line in `CHANGELOG.md` with the file and line it came from. Clean-ups that come with the rewrite (dead code, accessibility, labels) are listed in the module specification and not in the changelog.

## Tests

Everything runs with one command, `uv run pytest`, from `02_backend/`. There is no JavaScript test runner, because the frontend has no Node tooling. Chromium for the browser tests is installed once with `uv run playwright install chromium`.

Backend tests call the API through FastAPI's test client: save, list, load, overwrite and delete a project; upload and download a sample; reject a file that is too large or not audio; reject a project that references an unknown sample.

Browser tests use Playwright for Python. A pytest fixture starts the backend on a free port, so the tests need no running container. They come in three kinds:

- Module checks import the real JavaScript modules inside the page and call them, for example the knob taper, the WAV encoder or the scale tables. This replaces JavaScript unit tests.
- Live checks open the page with `?test=1`. In that mode the page exposes `window.__rabbit` with the engine, the transport and a log of every `onStep` call (instrument, step, time). The tests read the log to prove that two instruments got the same step at the same audio time, that a tempo change reached every instrument, and that stop left no voice sounding.
- Offline checks render a few bars in an `OfflineAudioContext` and compare step times and signal level, so no test depends on a loudspeaker.

Two things stay manual and are written into the stage where they apply: the listening comparison with the original file, and the check that the click keeps time while the browser tab is hidden.

Each module specification ends with the list of tests for that module. Where a specification names a test file, the file is a pytest file in `02_backend/tests/`.

## Stages

Each stage is one pull request, adds one thing and ends with something a person can try. This is the rule the workshop teaches: one feature at a time. The column Status is updated in the pull request that finishes the stage.

| Stage | Adds | Done when | Status |
|---|---|---|---|
| 0 Skeleton | The container, the backend with `/api/health` and the frontend route, an empty page, the test setup with one backend test and one browser test. Replaces the template's Docker files. | `docker compose up --build` works on a clean machine, the page opens on `http://localhost:8080`, and `uv run pytest` passes. | open |
| 1 Design system | Tokens, fonts, chassis, the shared `ui/` components, the shell with header and tab row. | A demo page shows every component in the RABBIT look, usable with the keyboard. | open |
| 2 Engine and Metronome | Engine, mixer, transport with Worker clock, `fx/reverb.js`, `fx/delay.js`, the store with the instrument switches, the Metronome as first instrument. | The click follows start, stop and tempo. Manual check: it keeps time while the browser tab is hidden. | open |
| 3 Projects and samples | The project and sample routes of the backend, save and load in the store. | A saved project has the same settings and the same step times after a reload. | open |
| 4 Beat | The Beat module. | Every row of its parity table is ticked. Beat and Metronome get the same step at the same audio time. Manual check: listening comparison with the original Beat Machine. | open |
| 5 Synth | The Synth module. | Every row of its parity table is ticked. | open |
| 6 Console | The Console interface for the strips, the WAV tracks and their timeline. | Every row of its parity table is ticked, except the mixdown rows, which belong to stage 9. Mute and solo act on instruments and WAV tracks. | open |
| 7 Slicer | The Slicer module. | Every row of its parity table is ticked. A loaded sample is part of the saved project. | open |
| 8 Theremin | The Theremin module with camera access. | Every row of its parity table is ticked. The frame analysis is tested on a synthetic image; the camera path is tested with Chromium's fake camera. | open |
| 9 Mixdown | WAV export of WAV tracks and instruments. | The exported file contains every instrument that was switched on, except the Theremin. | open |
| 10 Release | Screenshot, user guide, parity review against `03_originals/`. | A person who did not build it runs it from the README and finds every function of the originals. | open |

Stages 0 to 3 build on each other. Stages 4 to 8 depend only on stage 3, so different people can work on them at the same time. Stage 9 needs stages 4 to 7.

## Risks and open points

- The inventory and the specifications come from reading the source code. Nobody ran the original files to confirm the described behaviour. Each instrument stage contains a listening comparison with its original.
- The original files carry no licence. Tom Tiest's agreement to publication under the MIT licence of this repository must be on record before stage 0 is merged.
- The RABBIT logo is embedded in the original files as image data. It is used with Tom Tiest's permission and is not part of the HPI corporate design.
- Michroma and Barlow are published under the SIL Open Font Licence, which allows serving them from the container. The licence texts go into `01_frontend/fonts/`.
- A Web Worker timer is not slowed down like a page timer in current browsers, but the scheduling itself still runs on the page. A page that is busy drawing can still be late; the transport then skips steps and does not play them in a burst.
- [open-questions.md](open-questions.md) lists the questions the specifications raise. The ones marked for the owner are answered before the stage of that module starts.
