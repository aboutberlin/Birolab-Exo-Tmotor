

## Table of Contents

- [System Overview](#system-overview-real-time-torque-control-architecture-for-hip-exoskeleton)
- [Hardware Components](#hardware-components)
- [System Control & Communication Flow](#control--communication-flow)
- [System Detailed Workflow](#detailed-workflow)
- [System Deployment Overview](#deployment-notes)
- [Requirement Py Package](#Requirement-package)
- [How to Deploy Code to the Raspberry Pi](#how-to-deploy-code-to-the-raspberry-pi)
- [How to Deploy Code to the Teensy 41](#how-to-deploy-code-to-the-teensy-41)
- [How to Deploy Code to the Adafruit Itsy-Bitsy nRF52840 (Central & Peripheral)](#how-to-deploy-code-to-the-adafruit-itsy-bitsy-nrf52840-central--peripheral)
- [How to Run the GUI](#how-to-run-the-gui)
- [Eigen Path Configuration (for C++ Compilation)](#eigen-path-configuration-for-c-compilation)



## System Overview: Real-Time Torque Control Architecture for Hip Exoskeleton

This codebase represents the **post-training deployment phase** of the reinforcement learning (RL) pipeline. The RL model has already been trained, and the resulting `.pt` or neural network (`.nn`) files are ready for direct deployment on the physical system. No additional training is required—only model loading and real-time execution.

---

## Hardware Components

| Component                         | Description                                                                |
|----------------------------------|----------------------------------------------------------------------------|
|  **Computer**                  | Loads GUI interface for visualization and user control.                    |
|  **Adafruit Itsy-Bitsy nRF52840 (central)** | Acts as Bluetooth central module, connected to PC via USB-C. |
|  **Adafruit Itsy-Bitsy nRF52840 (peripheral)** | Soldered to PCB, communicates wirelessly with central.          |
|  **Teensy 4.1**                    | Low-level controller; communicates with Raspberry Pi and actuates motor.  |
|  **Raspberry Pi**              | Hosts RL controller, computes torque based on IMU input.                  |
|  **Motor (Various)**                     | Applies torque to assist motion.                                          |
|  **IMU**                       | Sends real-time joint angle and velocity data.                            |

---

## Control & Communication Flow

![System Flow Diagram](image/image1.png)
---

## Detailed Workflow

1. **Computer loads GUI**, providing visualization and interaction for users.
2. The GUI communicates with the **central Bluetooth module (nRF52840)** via USB-C.
3. The central Bluetooth talks to a **peripheral nRF52840**, soldered to the main PCB.
4. The peripheral forwards data through **serial communication** to the **Teensy**.
5. **Teensy receives** GUI hyperparameters and **IMU data**, then:
    - Passes the IMU data to **Raspberry Pi** for inference.
    - Receives torque prediction from the Raspberry Pi.
    - Sends torque commands to the **motor driver**.
6. The motor applies torque to assist motion, and sensor feedback is returned.
7. Torque and IMU feedback are sent back through the same chain to the GUI for display.

## Deployment Notes

- **Reinforcement Learning Controller (Raspberry Pi)**  
  The trained RL model (`.pt` file via PyTorch) is deployed on a **Raspberry Pi**. The following Python scripts under `/Higher_level_controllers` are responsible for high-level control:
  - `DNN_torch.py` – Loads the trained PyTorch model.
  - `RL_controller_torch.py` – Handles real-time control and torque prediction.
  - `ReadIMU.py` – Reads IMU sensor data (angles, angular velocity) from the serial port.

- **Low-Level Controller (Teensy)**  
  The **Teensy** microcontroller runs firmware written in Arduino (C++), which receives torque commands from the Raspberry Pi and drives the motors accordingly.  
  The `/Lower_level_controllers/` directory contains multiple subfolders, each corresponding to different motor drivers or exoskeleton configurations.  
  - **Current configuration**: We are using the `Hip_V15` directory, which should be compiled and flashed to the Teensy via the Arduino IDE.

- **Graphical User Interface (PC)**  
  The GUI is implemented in Python and supports real-time plotting of sensor data and torque feedback.  
  - Located under `GUI/0-RL/`, main scripts include:
    - `UF_GUI_Improved_v2.py` – The enhanced GUI interface.
    - `Motor_test_gui.py` – GUI version with motor testing capabilities.

- **Closed-loop Feedback System**  
  The entire torque control system operates in a **closed-loop** manner, using real-time IMU feedback to adjust torque predictions and motor commands.


## Requirement Package

This project requires several Python dependencies. All required packages are listed in `requirements.txt` for easy installation.  
It is **recommended to use Python 3.9**, preferably within a virtual environment (e.g., `conda`).

### Recommended Setup (with `conda`)

```bash
# 1. Create a new Python 3.9 environment
conda create -n hip_exo_env python=3.9
conda activate hip_exo_env

# 2. Install all required packages
pip install -r requirements.txt
```

### `requirements.txt` Contents

```txt
# GUI and plotting
PyQt5==5.15.9
pyqtgraph==0.13.3

# Serial communication
pyserial==3.5

# Scientific computing
numpy==1.24.4
torch==2.0.1

# Optional (if used in the future)
# scipy==1.10.1
```

---

## How to Deploy Code to the Raspberry Pi

![System Flow Diagram](image/image2.PNG)

To deploy the reinforcement learning controller to the Raspberry Pi, follow the setup below:

### 1. Physical Connection

- Use a **mini HDMI to HDMI cable** to connect the Raspberry Pi to a monitor. This provides screen output and network configuration capabilities.
- Connect the Raspberry Pi to your **local network** to ensure SSH access.

### 2. Remote Code Deployment via VSCode

- Use **VSCode Remote SSH** to connect to the Raspberry Pi from your PC.
- Upload the required code (see below) to the Raspberry Pi via the remote VSCode interface.
- You can then run the scripts directly on the Raspberry Pi terminal within VSCode.

### 3. Code Files to Upload and Run

The following Python scripts should be deployed to the Raspberry Pi:

- `DNN_torch.py` – Loads the trained PyTorch model.
- `RL_controller_torch.py` – Handles real-time torque prediction and control logic.
- `ReadIMU.py` – Continuously reads IMU data (e.g., joint angles and angular velocity) via the serial interface.

### 4. Data Flow Summary

- The **Raspberry Pi** connects to a **monitor** via HDMI to enable direct access and debugging.
- The **PC** connects to the Raspberry Pi over SSH using **VSCode Remote** for code editing and execution.
- The deployed controller scripts read sensor input from the IMU and send predicted torque commands to the Teensy via serial communication.


---

## How to Deploy Code to the Teensy 4.1

1. **Install the Arduino IDE**  
   Download and install the appropriate version of the Arduino IDE from the official website:  
   [https://www.arduino.cc/en/software](https://www.arduino.cc/en/software)

2. **Install Teensy Support**  
   Follow the setup instructions provided on the PJRC official page:  
   [https://www.pjrc.com/teensy/td_download.html](https://www.pjrc.com/teensy/td_download.html)

3. **Install Required Libraries**  
   In the Arduino IDE, open the **Library Manager**, search for `"MovingAverager"` by Ian Carey, and install it.

---

## How to Deploy Code to the Adafruit Itsy-Bitsy nRF52840 (Central & Peripheral)

### Bluetooth Setup

You will need **two Adafruit Itsy-Bitsy nRF52840** modules to enable wireless serial communication between the exoskeleton and the host PC.

- The **peripheral device** (soldered onto the PCB) should be flashed with the following code:  
  [Peripheral Code – BLE PRPH 30 Data](https://github.com/biomechatronics001/Hip_Exoskeleton_v1.4_Control_Software/tree/main/3.%20Bluetooth%20code%20for%20ItsyBitsy%20wireless%20board%20in%20exoskeleton%20electronics%20board/High_speed_ble_prph_30data)

- The **central device** (connected to the PC) should be flashed with:  
  [Central Code – BLE Central 30 Data](https://github.com/biomechatronics001/Hip_Exoskeleton_v1.4_Control_Software/tree/main/4.%20Bluetooth%20code%20for%20ItsyBitsy%20wireless%20board%20on%20high%20level%20computer/High_speed_ble_central_30data)

> **Note:** When uploading code to the nRF52840, you may need to install additional board support and drivers:
- For **Arduino IDE setup and board configuration**:  
  [https://learn.adafruit.com/adafruit-itsybitsy-nrf52840-express/arduino-support-setup](https://learn.adafruit.com/adafruit-itsybitsy-nrf52840-express/arduino-support-setup)

- If you are **unable to open the serial port (Linux)**, consider updating the bootloader:  
  [https://learn.adafruit.com/adafruit-itsybitsy-nrf52840-express/update-bootloader-use-arduino-ide](https://learn.adafruit.com/adafruit-itsybitsy-nrf52840-express/update-bootloader-use-arduino-ide)

---

## How to Run the GUI

### Requirements

1. The Teensy firmware must be running and capable of collecting, encoding, and transmitting sensor data over serial communication.
2. The Itsy-Bitsy nRF52840 Bluetooth modules (both peripheral and central) must be properly flashed and connected.
3. The host PC must have the required Python libraries installed.

```bash
pip install pyqtgraph pyserial PyQt5
```

Or use the prepared `requirements.txt` with:

```bash
pip install -r requirements.txt
```

### Running the GUI

1. Connect the **central Bluetooth device (nRF52840)** to your PC via USB.
2. Run the latest GUI script using Python:
   ```bash
   python GUI/0-RL/HipExo_GUI_v11.py
   ```
3. In the GUI:
   - Select the correct COM port.
   - Click the **"Connect Bluetooth"** button.
   - The data will begin streaming in real time.

### GUI Features

- Real-time visualization of 6 different signals:
  - Left IMU pitch angle
  - Right IMU pitch angle
  - Left motor: desired and actual torque
  - Right motor: desired and actual torque
- Automatically detects available COM ports
- Allows COM port selection and Bluetooth pairing
- Data logging to CSV file for offline analysis

> Sample Bluetooth MAC addresses:  
> `IMU1: 3B:04:94:38:88:F6`  
> `IMU2: E9:65:F2:E3:6E:58`

### Reference

- YouTube demo video of GUI: [https://youtu.be/w6m0pwIsVQM](https://youtu.be/w6m0pwIsVQM)

---

## Eigen Path Configuration (for C++ Compilation)

If compiling C++ code (e.g., `Pre_intention.cpp`) that uses the Eigen library, ensure that the path to Eigen is correctly specified.

Example compile command on macOS (Homebrew installed Eigen):

```bash
clang++ -std=c++17 -I /opt/homebrew/include/eigen3 -o main Pre_intention.cpp
```
