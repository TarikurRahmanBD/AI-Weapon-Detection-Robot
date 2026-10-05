# AI Weapon Detection Robot

A Python and Flask application that streams camera video to a browser dashboard, runs weapon/person detection with Ultralytics YOLO, records detection snapshots and events, and offers manual rover and arm controls over an Arduino serial connection.

> This is an experimental computer-vision and robotics project. Detection can be inaccurate and must not be used as the sole basis for safety, security, or law-enforcement decisions. Detection does not automatically move or actuate the robot; movement commands are issued through the dashboard controls.

## Features

- Live MJPEG camera feed with annotated detections and adjustable confidence threshold.
- Detection event list, CSV export, and locally saved snapshots (`snapshots/`).
- Browser dashboard with normal, night, thermal, and grayscale display modes.
- Optional GPS display and alert markers when the browser provides location access.
- Manual rover, speed, and robotic-arm controls through Arduino serial.
- Uses `weapon_detection_yolov12.onnx` when present and falls back to `weapon_detection_yolov12.pt`.

## Requirements

- Python 3.9 or newer.
- Linux is the intended runtime, especially Raspberry Pi OS. The camera code looks for Linux USB cameras using V4L2 and then tries Raspberry Pi Camera Module support through Picamera2. Generic Windows/macOS camera discovery is not implemented.
- A supported camera for video detection.
- Arduino-compatible board and matching motor/servo hardware for physical control (optional).
- `git` to clone the repository. `run.sh --https` additionally requires Bash and OpenSSL.

## Installation

Clone the repository and create an isolated Python environment:

```bash
git clone https://github.com/TarikurRahmanBD/AI-Weapon-Detection-Robot.git
cd AI-Weapon-Detection-Robot
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install flask ultralytics opencv-python numpy onnxruntime pyserial gunicorn
```

On Raspberry Pi OS, install and configure the camera support required by your camera. For a Pi Camera Module, install the system's Picamera2 package using the Raspberry Pi OS instructions; Picamera2 is only needed for that camera type.

## Run

For a quick local run (HTTP):

```bash
python app.py
```

Open `http://localhost:5000`. To access the dashboard from another device, use the host machine's LAN IP and make sure both devices are on the same network.

For the Gunicorn runner on Linux:

```bash
chmod +x run.sh
./run.sh
```

The script serves HTTP on port `5000` by default. To generate/use a self-signed certificate and serve HTTPS:

```bash
./run.sh --https
```

The browser will warn about the self-signed certificate. GPS access in browsers generally requires HTTPS or localhost. `run.sh` is a Bash/Linux script and does not run natively in Windows PowerShell.

## Arduino Setup

1. Open `arduino.cpp` in the Arduino IDE, install the `Servo` library if needed, select the board, and upload the sketch.
2. Wire the motor driver and servos to the pins defined near the top of `arduino.cpp`. Confirm your wiring and motor direction with the robot lifted off the ground before driving it.
3. Connect the Arduino to the host over USB. The app attempts to detect common Arduino/CH340 serial devices automatically. If needed, set `ARDUINO_PORT` before starting the app:

	Linux:

	```bash
	export ARDUINO_PORT=/dev/ttyUSB0
	python app.py
	```

	Windows PowerShell (for serial configuration):

	```powershell
	$env:ARDUINO_PORT = "COM5"
	python app.py
	```

The serial baud rate is `9600`. The firmware accepts these movement commands: `F` forward, `B` backward, `R` left, `L` right, `W` stop, and `G`/`H`/`I`/`J` for diagonal movement. Arm commands use `ARM:<base>,<elbow>,<gripper>` with each value from 0 to 180; drive speed uses `SPD:<0-255>`. If no Arduino is connected, video detection still works and serial controls report disconnected.

## Models and Configuration

The application checks for model files in this order:

1. `weapon_detection_yolov12.onnx`
2. `weapon_detection_yolov12.pt` if the ONNX file is absent

`yolo12n.pt` is included as a separate base model and is not selected by the application automatically. Detection labels depend on the model's trained class names. The confidence threshold can be changed in the dashboard.

## Security and Privacy

- Do not expose this development server directly to the public internet. Use it only on a trusted network unless you add proper authentication, authorization, and deployment hardening.
- The dashboard's control unlock password is currently hard-coded as `1234` in `app.py`. It is not secure authentication, and the control endpoints do not require that unlock. Change/remove this behavior before connecting the robot to an untrusted network.
- HTTPS is optional; plain HTTP is the default. A self-signed certificate encrypts traffic but does not establish trusted server identity.
- Camera frames are streamed to connected clients. Detection snapshots are written under `snapshots/` and remain on the host unless separately shared.
- Detection may produce false positives or miss objects. Keep physical controls under direct human supervision.

## Troubleshooting

- **No video:** Confirm a supported camera is connected and accessible. On Raspberry Pi, verify camera configuration and Picamera2 installation. The camera starts when a dashboard viewer connects.
- **Model load error:** Confirm that at least one of the two application model files exists and that the Python dependencies installed successfully.
- **Arduino offline:** Check the USB cable, serial permissions, selected port, and that no other program has opened the port. Set `ARDUINO_PORT` explicitly if auto-detection fails.
- **Port already in use:** Stop the existing process using port `5000`, or change the port in `run.sh` and the corresponding app configuration.
- **Browser GPS unavailable:** Use HTTPS or open the app on localhost; accept the self-signed certificate warning when using the HTTPS runner.

## Repository Files

| File | Purpose |
| --- | --- |
| `app.py` | Flask dashboard, camera capture, YOLO inference, snapshots, and serial command endpoints |
| `arduino.cpp` | Arduino firmware for rover motors and arm servos |
| `weapon_detection_yolov12.onnx` | Preferred ONNX detection model |
| `weapon_detection_yolov12.pt` | PyTorch model fallback |
| `yolo12n.pt` | Additional base YOLO model; not selected automatically |
| `run.sh` | Linux/Bash Gunicorn launcher with optional HTTPS |
| `cert.pem`, `key.pem` | Self-signed TLS files generated by `run.sh --https` when absent |

## Developer

**Tarikur Rahman**  
GitHub: [@tarikurrahmanbd](https://github.com/tarikurrahmanbd)  
Portfolio: [yourtarikur.netlify.app](https://yourtarikur.netlify.app/)  
Email: [tarikurrahman08@gmail.com](mailto:tarikurrahman08@gmail.com)

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).
