<div style="background-color: #ffffff; color: #000000; padding: 10px;">
<img src="00_aisc/img/logo_aisc_bmftr.jpg">
<h1>RABBIT Studio</h1>
</div>

RABBIT Studio puts the browser instruments of the musician and teacher [Tom Tiest](https://www.iamarabbit.com/) into one application that runs in one Docker container. Tom built each instrument as a separate HTML file with an AI chatbot. This repository rebuilds them as one studio with one shared clock, as a teaching example for the workshop series [Rapid Prototyping via Vibe Coding](https://github.com/aihpi/workshop-rapid-prototyping).

Status: planning. The application is not built yet. The plan will be added to this repository before the first line of application code.

![Screenshot of the application](00_aisc/img/screenshot_app.png)

The screenshot is the placeholder of the template. It will be replaced when the application runs (TBD).

## Features

The studio will hold six instruments, each with every function of the HTML files it comes from.

- A beat machine with a step sequencer, drum voices and sample loading.
- A synth with a piano roll, a bass sequencer, filter and envelope.
- A slicer that cuts a sample into slices and plays them from a sequencer.
- A theremin that turns webcam input into pitch.
- A metronome with a slot machine for random values.
- A mix console with channels, equaliser, reverb send and a master bus.

All instruments follow one transport, so they play in time without the wall-clock trick of the original files.

## Setup and Installation

### Prerequisites

- Docker and Docker Compose. The repository [workshop-getting-started](https://github.com/aihpi/workshop-getting-started) explains how to install them.

### Quick Start

1. Clone the repository:
   ```bash
   git clone https://github.com/aihpi/demo-rabbit-studio.git
   cd demo-rabbit-studio
   ```

2. Start the application: TBD, the application is not built yet.

3. Access the application: TBD.

## User Guide

### Using the Tool

TBD, follows with the application.

### Recommendations

TBD.

## Limitations

- The repository holds no application code yet.
- The studio runs on one computer. It does not synchronise several computers over the network.

## References

- [Tom Tiest, I Am A Rabbit](https://www.iamarabbit.com/)
- [Workshop Rapid Prototyping via Vibe Coding](https://github.com/aihpi/workshop-rapid-prototyping)
- [Web Audio API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API)

## Author

- [Hanno Müller](https://hpi.de/kisz)
- Instruments and design by [Tom Tiest](https://www.iamarabbit.com/)

## Issues and the project board

`.github/workflows/add-issue-to-project.yml` adds every new issue to the [AIHPI project board](https://github.com/orgs/aihpi/projects/3). It needs a token in the secret `ADD_ISSUE_TO_PROJECT`, because the workflow's own `GITHUB_TOKEN` cannot write to organisation projects. This repository is public, so it inherits the organisation-level secret and nothing needs to be set.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).

---

## Acknowledgements
<img src="00_aisc/img/logo_bmftr_de.png" alt="drawing" style="width:170px;"/>

The [AI Service Centre Berlin Brandenburg](http://hpi.de/kisz) is funded by the [Federal Ministry of Research, Technology and Space](https://www.bmbf.de/) under the funding code 16IS22092.
