# Parkinsons Motor Severity Monitoring

A comprehensive IoT and Machine Learning system designed to continuously monitor and assess the motor severity of Parkinson's disease patients. 

## Overview

This project integrates edge devices (sensors) with a robust backend architecture to provide real-time data streaming and motor severity classification. It uses a Temporal Convolutional Network (TCN) model for precise analysis of sequential sensor data.

## Architecture

The repository is structured into the following core components:

- **`sensor/` & `sensor_ws/` (Firmware)**: Contains the C/C++ codebase for the hardware sensors (e.g., ESP32/Arduino). Responsible for data acquisition and transmitting telemetry data via WebSockets for low-latency streaming.
- **`backend/` (Server & Machine Learning)**: A Python-based backend that handles WebSocket connections, processes incoming data, and runs the TCN (Temporal Convolutional Network) model for severity prediction.
- **`firebase/`**: Contains configurations and scripts for integrating with Firebase for real-time database capabilities, user authentication, and data persistence.

## Tech Stack

- **Hardware/IoT:** C/C++, Arduino IDE, ESP32, WebSockets
- **Backend:** Python, Flask, WebSockets
- **Machine Learning:** Temporal Convolutional Networks (TCN), SciPy, Keras/TensorFlow
- **Cloud/Database:** Firebase

## Getting Started

### Prerequisites
- Python 3.8+
- Arduino IDE (for flashing sensor firmware)
- Firebase Account

### Backend Setup
1. Navigate to the backend directory:
   ```bash
   cd backend
   ```
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```
3. Run the backend server:
   ```bash
   python app_ws.py
   ```

### Firmware Setup
1. Open the `sensor_ws/sensor_ws.ino` file in the Arduino IDE.
2. Update the Wi-Fi credentials and WebSocket server IP address.
3. Flash the code to your ESP32 or compatible microcontroller.

## License

This project is licensed under the MIT License - see the LICENSE file for details.
