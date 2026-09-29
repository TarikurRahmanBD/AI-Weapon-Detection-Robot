# 🔫 AI-Powered Weapon Detection & Surveillance Robot

An advanced, real-time security and surveillance robot system integrating state-of-the-art computer vision with hardware automation. This project utilizes custom-trained **YOLOv12** models to detect weapons in real-time and communicates with an **Arduino**-controlled robotic platform for active response and surveillance.

---

## 👤 Developer & Creator
*   **Lead Developer & Inventor:** **Tarikur Rahman**
*   **GitHub:** [@tarikurrahmanbd](https://github.com/tarikurrahmanbd)
*   **Portfolio:** [yourtarikur.vercel.app](https://yourtarikur.vercel.app)

---

## 🚀 Features

*   **Next-Gen Detection:** Powered by the ultra-fast **YOLOv12** deep learning model (ONNX and PyTorch formats included).
*   **Secure Streaming:** Integrated HTTPS communication using local SSL certificates (`cert.pem`, `key.pem`) for secure web-based surveillance.
*   **Hardware Integration:** Seamless serial/wireless communication with **Arduino (C++)** to trigger alarms or control robotic movement (pan/tilt, chassis drive) upon weapon detection.
*   **One-Click Boot:** Quick startup shell script (`run.sh`) for rapid deployment.

---

## 🛠️ Tech Stack & Tools

*   **AI & Computer Vision:** Python, PyTorch, ONNX, OpenCV, Ultralytics YOLOv12
*   **Microcontroller / Firmware:** C++, Arduino IDE
*   **Networking & Security:** HTTPS (SSL/TLS), OpenSSL
*   **Scripting:** Bash (Shell Scripting)

---

## 📂 Repository Structure

```text
├── app.py                         # Main Python application (YOLO inference & server)
├── arduino.cpp                    # Firmware for microcontrollers (motors, sensors, alarms)
├── weapon_detection_yolov12.pt    # PyTorch weights for custom weapon detection
├── weapon_detection_yolov12.onnx  # Optimized ONNX model for high-speed inference
├── yolo12n.pt                     # Base YOLOv12 nano weights
├── run.sh                         # Bash script to automate setup and run the app
├── cert.pem / key.pem             # SSL certificates for encrypted communication
└── README.md                      # Project documentation
