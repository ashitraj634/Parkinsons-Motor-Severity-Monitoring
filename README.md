# Parkinson's Motor Severity Monitoring System

<div align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.8+-blue.svg" />
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-Deep%20Learning-ee4c2c.svg" />
  <img alt="ESP32" src="https://img.shields.io/badge/ESP32-Hardware-success.svg" />
  <img alt="Flask" src="https://img.shields.io/badge/Flask-Web%20Framework-000000.svg" />
</div>

<br/>

A real-time, hardware-accelerated machine learning platform designed to capture, isolate, and classify Parkinsonian tremors with high clinical accuracy. The system integrates low-latency IoT sensor arrays with a custom Temporal Convolutional Network (TCN) to provide orientation-invariant motion analysis. 

---

## 📖 Table of Contents
- [Problem Statement](#-problem-statement)
- [Technical Approach & Engineering Iterations](#-technical-approach--engineering-iterations)
- [System Architecture](#-system-architecture)
- [Setup and Usage](#-setup-and-usage)
- [License](#-license)

---

## 🛑 Problem Statement

Parkinson's disease and essential tremors present unique challenges in continuous monitoring. While clinical assessments are episodic and subjective, continuous wearable monitoring often fails due to three primary engineering challenges:

1. **The Orientation Problem:** As a patient moves their hand, the physical orientation of a wearable sensor changes. A tremor primarily affecting the X-axis might shift entirely to the Z-axis if the wrist is rotated. Traditional multi-channel neural networks often overfit to specific sensor orientations, leading to catastrophic failure during real-world inference.
2. **Signal Contamination:** Real-world tremors are not mathematically perfect sine waves. They exhibit "ramp-up" (initiation) and "ramp-down" (cessation) phases that contain heavily distorted frequencies. Training models on these boundary artifacts poisons the dataset.
3. **Latency vs. Accuracy:** Building a high-quality dataset requires waiting for a tremor episode to complete to analyze it. However, real-time clinical inference requires immediate detection while the tremor is still active.

---

## 🔬 Technical Approach & Engineering Iterations

This system was engineered through multiple iterations to solve these specific signal processing and machine learning bottlenecks.

### 1. Achieving Rotation Invariance (The 2-Channel Magnitude Architecture)

**Trial & Error:** Initial designs attempted to feed raw 6-channel data (3-axis Accelerometer, 3-axis Gyroscope) directly into the neural network. This resulted in a brittle model that failed when the sensor was worn at different angles. We considered data augmentation (channel permutation) to force the network to learn rotational invariance, but this required massive datasets and excessive compute resources.

**Final Solution:** We re-architected the feature extraction pipeline. Instead of feeding physical axes, the system computes the instantaneous 3D Vector Magnitude for both linear acceleration and angular velocity:
- **Channel 0 (Linear Motion):** `sqrt(Ax^2 + Ay^2 + Az^2)`
- **Channel 1 (Angular Motion):** `sqrt(Gx^2 + Gy^2 + Gz^2)`

By reducing the input to a 2-channel magnitude tensor, the model became mathematically immune to hand rotation. The TCN only learns the temporal "shape" of the tremor, yielding near-perfect classification with exceptionally small training datasets (as few as 10-15 samples per class).

### 2. Centralized Slice Extraction (Dataset Curation)

**Trial & Error:** Early implementations utilized a standard sliding-window approach (e.g., capturing the last 256 samples upon detecting a frequency spike). This approach frequently captured the chaotic "ramp-up" phase of a tremor, resulting in low-confidence training data.

**Final Solution:** We implemented an Active Episode state machine. Upon detecting a sustained dominant frequency in the 4-12 Hz band (verified via Fast Fourier Transform and dynamic peak prominence), the system enters an unbounded recording state. Once the tremor completely subsides, the algorithm evaluates the total episode length and surgically extracts exactly 256 samples from the absolute mathematical center of the recording. This guarantees that every training sample is 100% saturated with pure, stable tremor data, completely discarding boundary artifacts.

### 3. Decoupled Live Inference

**Trial & Error:** The Center Slice Extraction algorithm created a pristine dataset, but waiting for a tremor to finish before classifying it made the live inference dashboard feel unresponsive. 

**Final Solution:** We decoupled the inference trigger from the calibration trigger. The Digital Signal Processing (DSP) worker now maintains an asynchronous live buffer. If the system detects an uninterrupted tremor streak of 8 seconds (16 continuous FFT cycles), it instantly bypasses the episode lock, rips the most recent 256 samples from memory, and pushes them through the PyTorch model for immediate classification. The background episode continues to record undisturbed, satisfying both the need for instantaneous feedback and high-quality post-episode dataset generation.

---

## ⌚ Hardware Prototype

Here is the fully functional wearable sensor node built for this project. It features an ESP32 microcontroller, an MPU6050 6-axis IMU, a dedicated power switch, and a compact LiPo battery pack mounted on a standard wristband.

<div align="center">
  <img src="assets/hardware/esp32_sensor_node_top_view.jpg" width="30%" alt="ESP32 Sensor Node Top View" />
  <img src="assets/hardware/esp32_sensor_node_side_profile.jpg" width="30%" alt="ESP32 Sensor Node Side Profile" />
  <img src="assets/hardware/mpu6050_mounting_detail.jpg" width="30%" alt="MPU6050 Mounting Detail" />
</div>

---

## 🏗 System Architecture

- **Hardware Layer:** ESP32 Microcontroller + MPU6050 (6-DOF IMU) streaming via a local access point (`192.168.4.1`) over WebSockets at a strict 50Hz.
- **Signal Processing Layer:** Python backend utilizing `scipy.fft` for DC-offset removal, spectral analysis, and dynamic prominence peak finding to differentiate isolated vs. compound tremors. Includes Complementary Filters for accurate Pitch, Roll, and Yaw calculation via Alpha-blending.
- **Machine Learning Layer:** PyTorch Temporal Convolutional Network (TCN). Utilizes dilated causal convolutions (emulating ECAPA-TDNN depth styles) with strict causality enforced via custom `Chomp1d` layers and residual shortcut connections to extract rapid temporal features without looking ahead in the sequence.
- **Presentation Layer:** Flask + HTMX backend serving a responsive interface with live Chart.js spectral streaming and Three.js 3D orientation visualization.

---

## 🚀 Setup and Usage

### Prerequisites
- **Python 3.8+**
- **Arduino IDE** (Configured for ESP32 board management)

### Workflow

1. **Hardware Node:** Power the ESP32 to broadcast the local WebSocket stream.
2. **Launch Core:** Execute `python backend/app_ws.py` to start the signal processing server.
3. **Calibration:** Use the Calibration Dashboard at `http://localhost:5000/calibration` to record initial labeled samples using the Center Slice algorithm.
4. **Training:** Run `python backend/train.py` to auto-discover classes, compute magnitudes, and train the PyTorch TCN. Weights are automatically exported to the active model registry.
5. **Inference:** Navigate to the Inference Engine dashboard at `http://localhost:5000/inference` for live, hardware-accelerated classification of ongoing tremors.

---

## 📝 License

This project is licensed under the MIT License. See the `LICENSE` file for details.
