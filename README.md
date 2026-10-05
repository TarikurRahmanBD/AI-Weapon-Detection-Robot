# AI Weapon Detection Robot

<div align="center">

**A browser-based camera detection dashboard with optional Arduino rover and arm controls**

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python)](https://www.python.org/)
[![Ultralytics](https://img.shields.io/badge/AI-Ultralytics%20YOLO-blue)](https://github.com/ultralytics/ultralytics)
[![OpenCV](https://img.shields.io/badge/Video-OpenCV-green?logo=opencv)](https://opencv.org/)
[![Flask](https://img.shields.io/badge/Web-Flask-black?logo=flask)](https://flask.palletsprojects.com/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

[Overview](#project-overview) | [Features](#features) | [Quick Start](#quick-start) | [Dashboard Guide](#dashboard-guide) | [Arduino](#arduino-robot-controls) | [Troubleshooting](#troubleshooting)

</div>

---

## Table of Contents

- [Project Overview](#project-overview)
- [How It Works](#how-it-works)
- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Dashboard Guide](#dashboard-guide)
- [Detection and Model Selection](#detection-and-model-selection)
- [Arduino Robot Controls](#arduino-robot-controls)
- [HTTP Routes](#http-routes)
- [Snapshots, Events, and GPS](#snapshots-events-and-gps)
- [Configuration](#configuration)
- [Security, Privacy, and Responsible Use](#security-privacy-and-responsible-use)
- [Troubleshooting](#troubleshooting)
- [Frequently Asked Questions](#frequently-asked-questions)
- [Repository Structure](#repository-structure)
- [Contributing](#contributing)
- [About the Developer](#about-the-developer)
- [License](#license)

## Project Overview

AI Weapon Detection Robot combines camera-based object detection with a web dashboard and an optional Arduino-controlled robot platform. The Python application captures video, runs an Ultralytics YOLO model, annotates frames, and streams the result to a browser. The dashboard also displays detection status and offers operator-controlled rover and robotic-arm commands when an Arduino is connected.

The project is intended for learning, prototyping, and supervised experimentation with computer vision and robotics. It is not a certified security product. Model predictions can be wrong, and the robot does not automatically move when an object is detected.

### Intended workflow

1. Start the Flask application on a Linux host with a supported camera.
2. Open the dashboard from the host or another device on the same trusted network.
3. Inspect the live annotated feed and detection information.
4. Optionally connect an Arduino and use the dashboard controls to operate the rover and arm manually.
5. Review alert snapshots and export the in-memory event log when needed.

## How It Works

```text
Camera
  |
  v
OpenCV capture --> Background inference thread --> YOLO model
			   |                       |
			   +---- annotated frame <-+
				      |
				      v
			      Flask MJPEG stream
				      |
				      v
			      Browser dashboard

Operator controls --> Flask command route --> USB serial --> Arduino firmware
Browser location -------------------------------> event location markers
```

The camera/inference loop is shared by browser viewers rather than running a separate model pass for every connected client. Camera capture is lazy: it starts when a dashboard or video-feed viewer connects and is released after the configured viewer-idle interval.

Detection alerts and movement controls are separate. A detection can create an event and snapshot, but it does not send a drive or arm command to the Arduino.

## Features

### Live detection dashboard

- MJPEG camera feed with boxes and labels drawn over detected objects.
- Detection confidence threshold control in the browser.
- Current detection counts, alert state, and approximate inference FPS.
- Display transforms named Normal, Night, Thermal, and Gray. These alter the displayed image; they do not provide night-vision or thermal-camera sensing.
- Fullscreen viewing and a responsive dashboard layout.

### Event review

- Records an event when an alert changes from inactive to active.
- Saves an annotated JPEG snapshot for eligible alert events under `snapshots/`.
- Provides a browser snapshot gallery and a CSV download of the current event log.
- Keeps event data in application memory; it is not a permanent database.

### Optional location display

- Uses browser geolocation when permission is granted and the browser allows it.
- Associates the most recent available location with a new alert event.
- Displays current location and recent alert markers on the dashboard map.
- Location data is optional and may be unavailable on plain HTTP from a non-local host.

### Optional Arduino control

- Sends movement, speed, and arm commands over a serial connection.
- Attempts to find common Arduino and CH340 USB serial devices automatically.
- Supports manual connection retry from the dashboard.
- Continues to provide camera detection when no Arduino is connected; physical controls will not operate.

## Requirements

### Software

- Python 3.9 or newer.
- Linux is the intended runtime. The USB camera code uses Linux V4L2 device discovery; Raspberry Pi Camera Module support is attempted through Picamera2.
- Python packages: Flask, Ultralytics, OpenCV, NumPy, ONNX Runtime, PySerial, and Gunicorn for the `run.sh` launcher.
- Bash and OpenSSL for the optional `run.sh --https` mode.
- An internet connection may be needed during initial package installation or if a model dependency downloads additional assets.

### Hardware

- A USB webcam supported by V4L2, or a Raspberry Pi camera supported by Picamera2.
- For physical robot operation: an Arduino-compatible board, a compatible motor driver, motors, servos, power supply, and suitable wiring.

The Arduino and robot hardware are optional for running the detection dashboard. Hardware performance depends on the host, camera, model runtime, and power configuration; no minimum FPS is guaranteed.

### Platform note

The Python application may install on other operating systems, but camera discovery in the current implementation is designed for Linux and Raspberry Pi OS. The supplied `run.sh` is Bash-specific and is not a native Windows PowerShell launcher. Windows serial port examples are included below, but a supported Windows camera-capture path is not currently implemented.

## Installation

### 1. Clone the repository

```bash
git clone https://github.com/TarikurRahmanBD/AI-Weapon-Detection-Robot.git
cd AI-Weapon-Detection-Robot
```

### 2. Create and activate a virtual environment

Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Windows PowerShell:

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1
```

### 3. Install Python dependencies

There is no `requirements.txt` in this repository, so install the runtime packages directly:

```bash
python -m pip install --upgrade pip
python -m pip install flask ultralytics opencv-python numpy onnxruntime pyserial gunicorn
```

`pyserial` is optional for detection-only use. `gunicorn` is used by `run.sh`; the direct `python app.py` command uses Flask's built-in development server. If you do not use the ONNX model, ONNX Runtime may not be needed, but the included preferred model is an ONNX file.

### Raspberry Pi camera setup

For a Pi Camera Module, install Picamera2 using the instructions for your Raspberry Pi OS release. It is a system-level camera package and is not included in the Python package command above. For a USB webcam, verify that Linux exposes a V4L2 video device and that the application user has permission to access it.

## Quick Start

### Direct local run

Activate the virtual environment, then run:

```bash
python app.py
```

Open [http://localhost:5000](http://localhost:5000) on the host. The application prints startup information, selected model classes, serial connection status, and the local network address when available.

### Gunicorn run on Linux

```bash
chmod +x run.sh
./run.sh
```

The script starts Gunicorn on port `5000` and binds to all network interfaces. By default, it uses plain HTTP. To generate or reuse a self-signed certificate and run with HTTPS:

```bash
./run.sh --https
```

When using HTTPS, the browser will show a certificate warning because the certificate is self-signed. Only continue on a network and device you trust. Browser GPS APIs typically require HTTPS or a localhost origin.

To stop the server, press `Ctrl+C` in the terminal running it.

## Dashboard Guide

### Live feed and image modes

The main panel displays the video stream with model detections. Use the confidence slider to change the minimum confidence threshold. A higher value generally shows fewer detections; a lower value may show more uncertain predictions.

The image mode controls are:

| Mode | Effect |
| --- | --- |
| Normal | Shows the annotated camera image without an additional color transform |
| Night | Applies a green-tinted grayscale display effect |
| Thermal | Applies a color map to grayscale image intensity; it is not thermal sensing |
| Gray | Shows a grayscale display effect |

### Status and counts

The dashboard includes current weapon/armed/person-related counts, alert information, FPS, and serial status. Counts depend on the model's output class labels and the application's class matching rules. FPS is an approximate live processing rate, not a performance guarantee.

### Manual controls

The control panel is exposed through the dashboard's control unlock interface. Once available, the operator can use the on-screen drive control, speed slider, arm base slider, elbow buttons, gripper control, and home command. Keyboard drive controls are also shown in the dashboard.

The unlock interface is only a basic UI gate. It is not access control for the HTTP endpoints; see [Security, Privacy, and Responsible Use](#security-privacy-and-responsible-use) before making the server reachable by other devices.

## Detection and Model Selection

### Model files

At startup, `app.py` selects the first model path that exists:

1. `weapon_detection_yolov12.onnx`
2. `weapon_detection_yolov12.pt`, if the ONNX file is absent

If the selected model exists but cannot load, the application does not automatically retry with the other model. `yolo12n.pt` is included in the repository but is not selected by `app.py`.

### Classes and labels

The application reads the model's class names and matches a configured set of weapon-related labels, including `gun`, `pistol`, `rifle`, `knife`, and `weapon` (with some capitalization/plural variants). It also derives some person-related labels from combinations of person and weapon detections. Results therefore depend on the actual class names and training data in the selected weights; this is not a universal weapon detector.

### Inference behavior

The current application uses a relatively small input size and runs full inference on selected frames, reusing recent detections between inference frames to reduce workload. Capture is configured around `480 x 360` pixels. These are implementation choices intended to reduce CPU and memory traffic, especially on Raspberry Pi; they may affect detection quality and responsiveness.

Changing the dashboard confidence threshold affects which detections are considered. It does not retrain the model or change its learned classes.

## Arduino Robot Controls

### Upload the firmware

1. Open `arduino.cpp` in the Arduino IDE.
2. Install the Arduino `Servo` library if it is not already available.
3. Select the correct board and port, then compile and upload the sketch.
4. With the robot lifted clear of the floor, test the motor direction and stop behavior before operating it on the ground.

The comments in `arduino.cpp` describe the expected motor driver and pin assignments. Verify the wiring against your actual board and motor driver; do not rely on pin numbers alone to infer electrical compatibility.

### Pin assignments in the firmware

| Function | Arduino pin(s) |
| --- | --- |
| Motor driver enable | D2, D4, D8, D12 |
| Motor PWM | D5, D6, D3, D11 |
| Arm base servo | A0 |
| Arm elbow servo | A1 |
| Gripper servo | A2 |
| Serial baud rate | 9600 |

The firmware comments note that the Servo library uses Timer1, so motor PWM is assigned to other timer pins.

### Serial port selection

The app tries to detect common Arduino/CH340 devices. If automatic detection does not find the board, set `ARDUINO_PORT` before starting the application.

Linux example:

```bash
export ARDUINO_PORT=/dev/ttyUSB0
python app.py
```

Windows PowerShell example for serial selection:

```powershell
$env:ARDUINO_PORT = "COM5"
python app.py
```

The serial port must not be open in another application. Linux users may also need permission to access the serial device.

### Firmware command reference

| Command | Action |
| --- | --- |
| `F` | Forward |
| `B` | Backward |
| `R` | Turn left |
| `L` | Turn right |
| `W` | Stop motors |
| `G` | Back-left diagonal |
| `H` | Forward-left diagonal |
| `I` | Back-right diagonal |
| `J` | Forward-right diagonal |
| `U` / `D` | Move elbow up / down for a timed pulse |
| `O` / `C` | Open / close gripper for a timed pulse |
| `P` | Return arm to its home preset |
| `ARM:<base>,<elbow>,<gripper>` | Set arm servo values; each parameter is in the range 0-180 |
| `SPD:<value>` | Set drive PWM value in the range 0-255 |

The Arduino firmware includes a 500 ms movement watchdog that stops the motors when commands stop arriving. This is a firmware behavior, not a substitute for an emergency stop or safe hardware design. Test the full system with the wheels clear before use.

## HTTP Routes

The Flask application exposes these routes. They are intended for the local dashboard and trusted development use.

| Route | Method | Purpose |
| --- | --- | --- |
| `/` | GET | Dashboard page |
| `/video_feed` | GET | MJPEG camera stream |
| `/detections` | GET | Current detections, FPS, alert state, counts, and recent events |
| `/set_conf?value=0.35` | GET | Set the confidence threshold; values are clamped by the app |
| `/set_view?mode=night` | GET | Choose `normal`, `night`, `thermal`, or `gray` display mode |
| `/location` | POST | Accept browser latitude/longitude JSON |
| `/gps_markers` | GET | Return current location and recent alert markers |
| `/unlock` | POST | Check the dashboard's hard-coded UI unlock password |
| `/serial_status` | GET | Report serial connection status |
| `/serial_reconnect` | GET | Close and retry the serial connection |
| `/cmd?c=F` | GET | Validate and forward an Arduino command |
| `/gallery` | GET | Show saved detection snapshots |
| `/snapshots/<filename>` | GET | Serve a saved snapshot |
| `/events.csv` | GET | Download the current in-memory event list as CSV |

Important: the routes are not a hardened public API. In particular, `/cmd` does not require a successful `/unlock` request. Do not expose this application to an untrusted network without implementing real authentication and authorization.

## Snapshots, Events, and GPS

- The application creates the `snapshots/` directory beside `app.py` when it starts.
- An alert event is recorded on the transition from no alert to alert. Continuous detection during the same alert does not create a new event every frame.
- Events are kept in a bounded in-memory queue. They are not persisted after the Python process exits.
- When eligible, an annotated JPEG snapshot is saved with a timestamped `alert_...jpg` filename. The application applies a minimum time interval between snapshot writes.
- Alert markers are added only when a browser location has already been received. The app keeps a bounded recent marker list in memory.
- `/events.csv` exports the event queue currently in memory. It does not export every frame-level detection.

## Configuration

Several values are currently configured directly in `app.py` and `arduino.cpp`, rather than in a separate settings file.

| Setting | Current behavior/location |
| --- | --- |
| Web port | `5000`; in `run.sh` and the direct `app.run` call |
| Arduino port | `ARDUINO_PORT` environment variable; auto-detects when unset |
| Arduino baud rate | `9600` in `app.py` and `arduino.cpp` |
| Initial confidence | `0.35` in `app.py` |
| View modes | `normal`, `night`, `thermal`, `gray` |
| Camera capture size | `480 x 360` in `app.py` |
| Model input size | `192` in `app.py` |
| Snapshot folder | `snapshots/` beside `app.py` |
| Control UI password | Hard-coded as `1234` in `app.py`; not secure authentication |

Do not change serial baud rate on only one side: the host application and Arduino firmware must use the same value.

## Security, Privacy, and Responsible Use

- Do not deploy this development server on the public internet. The Gunicorn launcher binds to all interfaces, so devices on reachable networks may be able to access it.
- The dashboard password is hard-coded as `1234`. More importantly, control routes do not enforce this password. It is a UI lock, not authentication. Add proper server-side authentication and authorization before allowing untrusted clients to reach the app or robot.
- Plain HTTP is the default. HTTPS mode uses a self-signed certificate; it encrypts the connection but does not prove the server's identity to the browser.
- Camera video is sent to connected browsers. Snapshots are stored locally in `snapshots/`; protect, review, and delete them according to your privacy requirements.
- Browser location permission is optional. Location is associated with events only after the browser supplies it.
- Model detections can be false positives or false negatives and may vary by lighting, distance, occlusion, camera quality, and training data. Do not use this prototype as the sole basis for decisions affecting people.
- Operate motors and servos only under direct human supervision. Provide a physical emergency stop and validate the mechanical and electrical design independently.

## Troubleshooting

### The application reports that no camera is available

- Confirm that the camera is connected and visible to the operating system.
- For a Linux USB webcam, verify V4L2 access and device permissions.
- For a Raspberry Pi camera module, install/configure Picamera2 for the OS release in use.
- Open the dashboard to trigger camera startup. The capture loop is lazy and may release the camera after all viewers leave.
- Generic Windows/macOS webcam discovery is not implemented in this version.

### The selected model fails to load

- Confirm that `weapon_detection_yolov12.onnx` or `weapon_detection_yolov12.pt` exists beside `app.py`.
- The ONNX file is selected whenever it exists. If it is invalid or unsupported, the app does not automatically fall back to the `.pt` model; move/rename the ONNX file to test the PyTorch model.
- Check that Ultralytics and the runtime needed by the selected model are installed for the active Python environment.

### The Arduino status remains offline

- Check the USB cable, board power, and selected serial port.
- Close Arduino IDE Serial Monitor and other programs that may have opened the same port.
- Set `ARDUINO_PORT` explicitly and restart the Python process.
- On Linux, check user permissions for `/dev/ttyUSB*` or `/dev/ttyACM*` devices.
- Confirm the firmware was uploaded and both sides use `9600` baud.

### The dashboard loads but control commands do not move the robot

- Check the dashboard serial status and application terminal output.
- Verify the motor driver wiring, enable pins, external motor power, and ground connections.
- Test with the wheels lifted and review the command/pin mapping in `arduino.cpp`.
- Remember that AI detection does not issue movement commands automatically.

### The browser cannot access GPS

- Grant location permission to the browser.
- For access from another device, use HTTPS; many browsers allow geolocation on localhost but not on an insecure LAN HTTP origin.
- With the self-signed certificate, accept the browser warning only when you trust the host and network.

### Port 5000 is already in use

Stop the other server using the port or update the port consistently in `run.sh` and the direct-run section of `app.py`.

### Inference is slow or the stream stutters

- Close extra browser tabs and confirm that no other process is using the camera.
- The app already reduces camera resolution and runs inference on selected frames. These tradeoffs can affect both speed and detection accuracy.
- Raspberry Pi and other CPU-only systems may process more slowly than a machine with a supported accelerator. This project does not guarantee a particular FPS.

## Frequently Asked Questions

### Does the robot move automatically when a weapon is detected?

No. Detection updates the dashboard, event list, and eligible snapshot. Drive and arm commands are sent only through the manual controls or a valid command request.

### Can I run the detection dashboard without an Arduino?

Yes. The Arduino connection is optional. Camera capture and inference are separate from serial control.

### Does the thermal mode use a thermal camera?

No. It applies a color map to the grayscale camera image. It does not measure heat or infer temperature.

### Are events and snapshots saved permanently?

Snapshots are written to disk in `snapshots/`. The event list and GPS markers are in memory and are lost when the application process exits. The CSV route exports the current event list only.

### Can I use another trained model?

The app loads the ONNX file first, otherwise the PyTorch file. A replacement model should use class names compatible with the app's detection and label matching logic. A custom model may require changes in `app.py` and should be tested before use.

### Is HTTPS enabled by default?

No. `python app.py` and `./run.sh` use HTTP. On Linux, `./run.sh --https` enables HTTPS using a self-signed certificate.

## Repository Structure

```text
AI-Weapon-Detection-Robot/
|-- app.py                         # Flask dashboard, camera, inference, events, and routes
|-- arduino.cpp                    # Arduino motor and arm firmware
|-- weapon_detection_yolov12.onnx  # Preferred detection model
|-- weapon_detection_yolov12.pt    # Model used when ONNX file is absent
|-- yolo12n.pt                     # Additional model; not selected by app.py
|-- run.sh                         # Linux/Bash Gunicorn launcher
|-- cert.pem                       # Existing TLS certificate used by HTTPS launcher
|-- key.pem                        # Existing TLS private key used by HTTPS launcher
|-- snapshots/                     # Created at runtime for alert images
|-- LICENSE                        # MIT License
`-- README.md                      # Project documentation
```

The `run.sh` script generates a self-signed certificate only when HTTPS is requested and either certificate file is missing. Keep private key files private; do not publish keys intended for a real deployment.

## Contributing

Contributions that improve reliability, accessibility, documentation, testing, or hardware compatibility are welcome.

1. Fork the repository and create a focused feature branch.
2. Keep changes scoped and describe hardware or platform assumptions.
3. Test relevant behavior where possible. For hardware changes, document the board, wiring, and motor driver used.
4. Update this README when setup steps, controls, dependencies, or supported behavior changes.
5. Open a pull request with a clear summary and any remaining limitations.

## About the Developer

**Tarikur Rahman**

- GitHub: [@tarikurrahmanbd](https://github.com/tarikurrahmanbd)
- Portfolio: [yourtarikur.vercel.app](https://yourtarikur.vercel.app/)
- Email: [tarikurrahman2008@gmail.com](mailto:tarikurrahman2008@gmail.com)

## License

This project is distributed under the MIT License. See [LICENSE](LICENSE) for the full text.
