# Parkinson's Motor Severity Monitoring System

<div align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.8+-blue.svg" />
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-Deep%20Learning-ee4c2c.svg" />
  <img alt="ESP32" src="https://img.shields.io/badge/ESP32-Hardware-success.svg" />
  <img alt="Flask" src="https://img.shields.io/badge/Flask-Web%20Framework-000000.svg" />
</div>

<br/>

A comprehensive IoT edge-computing and Machine Learning architecture designed to continuously monitor, quantify, and assess the severity of motor tremors in patients with Parkinson's disease. By combining real-time Digital Signal Processing (DSP) with a custom deep Temporal Convolutional Network (TCN), this system delivers live severity inference with high precision.

---

## 📖 Table of Contents
- [System Overview](#-system-overview)
- [Key Features & Capabilities](#-key-features--capabilities)
- [Architecture details](#-architecture-details)
  - [Hardware (Edge) Layer](#hardware-edge-layer)
  - [Backend & DSP Layer](#backend--dsp-layer)
  - [Machine Learning Layer](#machine-learning-layer)
- [Getting Started](#-getting-started)

---

## 🔬 System Overview
Evaluating Parkinsonian tremor severity has traditionally relied on subjective clinical observations (e.g., UPDRS scales). This project introduces a fully quantitative, data-driven pipeline. An edge sensor package captures high-resolution 6-axis IMU data (accelerometer and gyroscope) at a strict 50Hz sampling rate, streaming it over low-latency WebSockets. The Python backend then processes this data using mathematical frequency domain analysis (FFT) to detect clinical episodes, subsequently feeding the isolated signal into a PyTorch-based Deep Learning model.

---

## ✨ Key Features & Capabilities

- **Low-Latency Edge Streaming:** Custom ESP32 C++ firmware acts as an autonomous WebSocket server, providing real-time data streaming over a local Hotspot without requiring external internet.
- **Automated Episode Extraction:** A dedicated Digital Signal Processing (DSP) thread runs real-time Fast Fourier Transforms (FFT) on a 256-sample sliding window to dynamically isolate active tremor episodes within the 4Hz–12Hz Parkinsonian frequency band.
- **Temporal Consistency Filtering:** Prevents false positives by enforcing multi-second streak thresholds before triggering a clinical episode state.
- **Deep Sequence Modeling:** Utilizes an ECAPA-TDNN inspired Temporal Convolutional Network (TCN). The network employs dilated convolutions to effectively capture long-range temporal dependencies in the time-series sensor data without suffering from vanishing gradients.
- **Interactive Calibration & Inference Dashboard:** Includes a modular Flask-based UI equipped with HTMX to monitor model status, visualize live data streams, and log custom datasets for active learning.

---

## 🏗 Architecture details

### Hardware (Edge) Layer
Located in `/sensor` and `/sensor_ws`.
- **Microcontroller:** ESP32 / Arduino environment.
- **Sensors:** 6-DoF IMU recording raw Acceleration (X,Y,Z) and Gyroscopic rotation (X,Y,Z).
- **Network:** Operates via its own access point (`192.168.4.1`) managing high-speed Websocket clients.

### Backend & DSP Layer
Located in `/backend` (`app_ws.py`).
- **Data Buffering:** Thread-safe memory locks managing sliding windows (256 frames).
- **Complementary Filters:** Calculates accurate Pitch, Roll, and Yaw using Alpha-blended (`alpha=0.96`) accelerometer and gyroscope integration.
- **Episode Detection:** Automatically slices datasets exactly at the mathematical center of a detected tremor episode for highest quality Machine Learning extraction.

### Machine Learning Layer
Located in `/backend` (`tcn_model.py`, `train.py`).
- **Model:** `TremorClassifierTCN` written in PyTorch.
- **Input Channels:** 2-channel temporal input consisting of Accelerometer Magnitude and Gyroscope Magnitude.
- **Structure:** Sequence of 3 Dilated Convolutional Blocks (`channels: [16, 32, 64]`) with strict causality enforced via custom `Chomp1d` layers and residual shortcut connections.

---

## 🚀 Getting Started

### Prerequisites
- **Python 3.8+**
- **Arduino IDE** (Configured for ESP32 board management)

### 1. Firmware Flashing
1. Connect your ESP32 microcontroller.
2. Open `/sensor_ws/sensor_ws.ino` in the Arduino IDE.
3. Flash the code to the board. The board will host a WiFi hotspot.
4. Connect your host machine to the ESP32's Hotspot network.

### 2. Backend Setup
1. Open a terminal and navigate to the backend directory:
   ```bash
   cd backend
   ```
2. Install the necessary Python packages:
   ```bash
   pip install -r requirements.txt
   ```
3. Run the Flask WebSockets server:
   ```bash
   python app_ws.py
   ```
4. Once running, visit `http://localhost:5000` in your browser to access the Monitoring Dashboard.

---

## 📝 License

This project is licensed under the MIT License. See the `LICENSE` file for details.
