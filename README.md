# Fire-Rhythms-

An interactive installation using Arduino and Max/MSP in which placing logs on a weight-sensitive fire pit triggers folk-story projections. Load cell data (via HX711) is sent to Max/MSP, which maps weight thresholds to dynamic, multi-projector video states.

**Project documentation:** [bellaciaramitaro.com/fire-rhythyms](https://www.bellaciaramitaro.com/fire-rhythyms) 

---

## Overview

The system works in two parts:

**Arduino (`weight-sensor.ino`)** reads from an HX711 load cell amplifier connected to a scale. It calibrates on startup using a known scale factor (`73.385`) and tares to zero, then continuously streams weight readings in grams over serial at 57600 baud, once per second.

**Max/MSP (`Weight_Sensor___Multiple_Projectors.maxpat`)** receives the serial stream, parses it into numeric weight values, and routes those values through a series of threshold comparisons. Each threshold — set at 200, 400, 600, 800, and 1000 units — triggers a distinct video state. The patch manages five video channels via `jit.playlist` players, each loaded with a looping media file. Video output is routed through `jit.gl.meshwarp` objects (tied to a shared `jit.world` render context named `Testing1`) for projection mapping and geometric correction. Each projector channel includes controls for grid dimensions, curvature, rotation, layer order, and the ability to save/load warp settings.

The conceptual design treats each added log to a fire as adding approximately 200 units of weight, so each log placed triggers a new projection state.

When participants place new logs of wood into the weight-sensitive fire pit, new folk story films are projected throughout the exhibition space. This installation serves as a prototype for interactive modes of storytelling. 

---

## Setup

### Hardware Requirements

- Arduino (Uno or compatible)
- HX711 load cell amplifier module
- Load cell / weight sensor
- USB cable for Arduino-to-computer serial connection
- Computer running Max/MSP 8.5+
- Projector(s) connected to the computer

### Wiring

Connect the HX711 to the Arduino as follows:

| HX711 Pin | Arduino Pin |
|-----------|-------------|
| DOUT      | Digital 2   |
| SCK       | Digital 3   |
| VCC       | 5V          |
| GND       | GND         |

### Arduino Setup

1. Install the [HX711 library by bogde](https://github.com/bogde/HX711) via the Arduino Library Manager.
2. Open `weight-sensor.ino` in the Arduino IDE.
3. If your scale requires a different calibration factor, replace `73.385` in `scale.set_scale(73.385)` with your own value. To find the correct value, follow the calibration procedure in the [HX711 library README](https://github.com/bogde/HX711).
4. Upload the sketch to your Arduino.
5. Open the Serial Monitor (set to **57600 baud**) to confirm weight readings are streaming correctly.

### Max/MSP Setup

1. Open `Weight_Sensor___Multiple_Projectors.maxpat` in Max/MSP 8.5 or later.
2. Ensure the **Jitter Tools** package is installed (required for `jit.gl.meshwarp` and `jit.world`).
3. **Connect to the correct serial port.** The patch uses a `serial` object configured for port `b` at 9600 baud. Update this to match the port your Arduino is connected to (e.g., port `a`, `c`, etc.). You can find available ports by sending a `print` message to the serial object.
4. **Relink the video files.** Each `jit.playlist` object references video files from a local path (`~/Desktop/IDA/INSTALL_PROJECTOR/`). Replace these with the correct paths to your own media files:
   - `fire dreams.mp4`
   - `circle running.mp4`
   - `bull narrative.mp4`
   - `bird narrative.mp4`
   - `another fire.MOV`
5. **Open the gates.** Each video channel uses a `onebang` gate that must be manually opened on first run. Click the bang button labeled "You need to bang here to open the gate" in each projector section before starting.
6. Turn on the toggle in the top-left of the patch to start the metro and begin polling the serial port.
7. Enable the `jit.world` render context by toggling it on (the toggle connected to `jit.world Testing1`).
8. Use the mesh warp controls in each projector section to align your projection surfaces. Hit **write** to save warp settings and **read** to reload them in future sessions.

### Weight Thresholds

The patch fires bangs at the following cumulative weight values, each triggering a new video state:

| Threshold | Triggered Action         |
|-----------|--------------------------|
| > 200     | Video state 1 (fire dreams) |
| > 400     | Video state 2 (circle running) |
| > 600     | Video state 3 (bull narrative) |
| > 800     | Video state 4 (bird narrative) |
| > 1000    | Video state 5 (another fire) |

Each threshold uses a `onebang` object so the trigger fires only once per crossing, with a delay before the gate resets (30 seconds for the final state, 6 minutes for others).
