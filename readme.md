# 🚤 Real-Time Propulsion Health Monitoring and Fault-Tolerant Control for an Autonomous Marine UAV

![STM32](https://img.shields.io/badge/MCU-STM32%20NUCLEO--C031C6-blue)
![PlatformIO](https://img.shields.io/badge/PlatformIO-Embedded-orange)
![Wokwi](https://img.shields.io/badge/Simulation-Wokwi-green)
![Language](https://img.shields.io/badge/Language-C%2FC%2B%2B-blue)
![Status](https://img.shields.io/badge/Status-Prototype-success)

## 📌 Project Overview

This project presents a **real-time propulsion health monitoring and fault-tolerant control system for an autonomous marine UAV**.

The system continuously monitors important propulsion parameters such as:

- ⚙️ Propulsion RPM
- ⚡ Motor current
- 🌡️ Motor temperature
- 📳 Mechanical vibration

An **STM32 NUCLEO-C031C6** acts as the main embedded controller. The sensor data is processed in real time and combined using a rule-based sensor-fusion approach to identify abnormal propulsion conditions.

The system classifies the propulsion condition into four levels:

```text
NORMAL → WARNING → CRITICAL → EMERGENCY

Depending on the detected condition, the controller automatically changes the propulsion response using a servo actuator, LEDs, and buzzer.
The complete prototype has been developed and validated using Wokwi simulation.
🎯 Objectives
The main objectives of this project are:
1. Monitor propulsion parameters in real time.
2. Detect abnormal propulsion operating conditions.
3. Identify propeller obstruction and overheating conditions.
4. Monitor dynamic vibration using an MPU6050.
5. Classify propulsion health into multiple severity levels.
6. Implement fault-tolerant propulsion control.
7. Record fault-event counts.
8. Calculate a simple propulsion health score.
9. Provide real-time visual and audible fault indication.
10. Develop an architecture that can later be extended to physical marine-UAV hardware.
🏗️ System Architecture
                 AUTONOMOUS MARINE UAV
                         │
                  PROPULSION UNIT
                         │
        ┌────────────────┼────────────────┐
        │                │                │
       RPM             CURRENT       TEMPERATURE
        │                │                │
        └────────────────┼────────────────┘
                         │
                      MPU6050
                     Vibration
                         │
                         ▼
              STM32 NUCLEO-C031C6
                         │
                REAL-TIME PROCESSING
                         │
                  SENSOR FUSION
                         │
                  FAULT CLASSIFIER
                         │
        ┌────────────────┼────────────────┐
        │                │                │
      NORMAL          WARNING       CRITICAL/EMERGENCY
        │                │                │
        └────────────────┼────────────────┘
                         │
                FAULT-TOLERANT CONTROL
                         │
          ┌──────────────┼──────────────┐
          │              │              │
         OLED        LEDs + Buzzer     Servo

🔧 Hardware Components
Component	Purpose
STM32 NUCLEO-C031C6	Main controller
KY-040 Rotary Encoder	Simulated propulsion RPM
Potentiometer 1	Simulated motor current
Potentiometer 2	Simulated motor temperature
MPU6050	Vibration/acceleration monitoring
SSD1306 OLED	Local system display
Green LED	Normal status
Yellow LED	Warning status
Red LED	Critical/Emergency status
Buzzer	Audible fault indication
Servo Motor	Simulated propulsion/throttle actuator


🔌 Pin Configuration
Component	Signal	STM32 Pin
KY-040	CLK	D2 / PA10
KY-040	DT	D3 / PB3
KY-040	SW	D4 / PB10
Current Potentiometer	Signal	A0 / PA0
Temperature Potentiometer	Signal	A1 / PA1
MPU6050	SDA	D14 / PB9
MPU6050	SCL	D15 / PB8
Green LED	Control	D5
Yellow LED	Control	D6
Red LED	Control	D7
Buzzer	Control	D8
Servo	Signal	D9
OLED	SDA	D14 / PB9
OLED	SCL	D15 / PB8


I²C Devices
Both the MPU6050 and OLED share the same I²C bus.
STM32 I²C Bus
     │
     ├── MPU6050 → 0x68
     │
     └── OLED    → 0x3C

⚙️ Fault Detection
The system uses multiple sensor parameters to identify propulsion faults.
1. Normal Operation
Temperature < 70°C
AND
Vibration < 300 mg
AND
No high-current obstruction condition

Response
STATUS : NORMAL
ACTION : MAINTAIN PROPULSION

- Green LED ON
- Buzzer OFF
- Servo = 90°
2. Propeller Obstruction
A propeller obstruction is identified using the relationship between RPM and motor current.
RPM < 30
AND
Current >= 6 A

Response
STATUS : WARNING
ACTION : REDUCE PROPULSION RPM

- Yellow LED ON
- Buzzer OFF
- Servo = 60°
This helps distinguish a mechanically loaded motor from simple low-speed operation.
3. Severe Propulsion Obstruction
A severe obstruction is detected when:
RPM < 15
AND
Current >= 8 A

Response
STATUS : EMERGENCY
ACTION : EMERGENCY THROTTLE REDUCTION

- Red LED ON
- Buzzer ON
- Servo = 0°
4. Motor Overheating
Temperature thresholds:
Temperature	Condition	Response
< 70°C	Normal	Maintain propulsion
70–79°C	Warning	Reduce propulsion load
80–89°C	Critical	Thermal protection
≥ 90°C	Emergency	Emergency shutdown


5. High Vibration
The MPU6050 provides acceleration measurements along three axes.
The current lightweight vibration indicator uses the change between consecutive acceleration samples:
ΔX = |Xcurrent - Xprevious|

ΔY = |Ycurrent - Yprevious|

ΔZ = |Zcurrent - Zprevious|

Vibration = ΔX + ΔY + ΔZ

The value is converted into an approximate mg-scale indicator.
Vibration >= 300 mg
        ↓
WARNING
        ↓
CHECK PROPELLER / BEARING

Note: This is a lightweight dynamic-change indicator for the prototype. It is not a laboratory-grade RMS vibration measurement.

🚦 Fault Priority
When multiple abnormal conditions occur simultaneously, the system applies the following priority:
1. Emergency Overheating
        ↓
2. Severe Propulsion Obstruction
        ↓
3. Critical Temperature
        ↓
4. Propeller Obstruction
        ↓
5. High Vibration
        ↓
6. Warning Temperature
        ↓
7. Normal

This ensures that high-severity faults receive the strongest response.
🛡️ Fault-Tolerant Control
The controller automatically changes the propulsion response based on system health.
State	Green	Yellow	Red	Buzzer	Servo
NORMAL	ON	OFF	OFF	OFF	90°
WARNING	OFF	ON	OFF	OFF	60°
CRITICAL	OFF	OFF	ON	ON	30°
EMERGENCY	OFF	OFF	ON	ON	0°


This converts the monitoring system into a fault-tolerant control system rather than simply a fault-detection system.
❤️ Propulsion Health Score
A simple health score from 0–100 is calculated.
Initial score:
100 / 100

Penalties are applied according to the detected condition.
Temperature
70–79°C  → -10
80–89°C  → -25
≥90°C    → -50

Vibration
≥300 mg → -15

Propeller Obstruction
Warning obstruction → -25
Severe obstruction  → -50

The score is bounded between:
0 ≤ Health Score ≤ 100

📊 Fault Event Monitoring
The system maintains event counters for:
Obstruction Events
Overheat Events
Vibration Events
Emergency Events

For example:
Vibration Events : 0

After detecting a vibration fault:
Vibration Events : 1

The counter does not continuously increase while the same fault remains active.
🖥️ Simulation
The project was developed using Wokwi to simulate the embedded hardware.
Simulated Inputs
- KY-040 → RPM
- Potentiometer → Motor current
- Potentiometer → Motor temperature
- MPU6050 → Vibration
Simulated Outputs
- OLED
- Green LED
- Yellow LED
- Red LED
- Buzzer
- Servo
This allows the complete control logic to be tested without requiring a physical marine UAV propulsion system.
🧪 Validation Results
The prototype was tested under multiple simulated conditions.
System Startup
MPU6050 : READY
OLED    : READY
SYSTEM READY

Normal Operation
Example:
RPM          : 0
Current      : 1.6 A
Temperature  : 39 C
Vibration    : 0 mg

STATUS       : NORMAL

High Vibration
The MPU6050 acceleration was changed dynamically.
Example:
Vibration : 2149 mg

STATUS : WARNING
ACTION : CHECK PROPELLER / BEARING

Vibration Recovery
After returning the simulated acceleration to a stable condition:
Vibration : 0 mg

STATUS : NORMAL

Fault Event Counter
Before fault:
Vibration Events : 0

After fault:
Vibration Events : 1

RPM Testing
The system successfully responded to different simulated encoder values, including:
6 RPM
9 RPM
12 RPM

Low RPM alone was not classified as an obstruction because the motor current remained low.
💻 Software
The project uses:
- C/C++
- Arduino Framework
- PlatformIO
- STM32 Platform
- Wokwi
Main Firmware
src/main.cpp

PlatformIO Configuration
platformio.ini

Wokwi Circuit
diagram.json

📁 Project Structure
Marine-UAV-Propulsion-Health-Monitoring/
│
├── README.md
├── LICENSE
├── platformio.ini
├── diagram.json
│
├── src/
│   └── main.cpp
│
└── docs/
    ├── Marine_UAV_Propulsion_Health_Monitoring_Report.pdf
    ├── Marine_UAV_Propulsion_Flowchart.pdf
    └── VALIDATION_TESTS.md

▶️ How to Run
Using PlatformIO
Step 1
Clone or download this repository.
Step 2
Open the project in Visual Studio Code with PlatformIO installed.
Step 3
Select:
STM32 NUCLEO-C031C6

Step 4
Build the project.
Step 5
Upload the firmware to the STM32 board.
Step 6
Open the serial monitor at:
115200 baud

Step 7
Observe:
RPM
Current
Temperature
Vibration
Health Score
Fault Counters
System Status

🌊 Industry Use Case
This architecture is intended as a prototype for applications such as:
- Autonomous marine UAVs
- Unmanned surface vehicles
- Marine inspection platforms
- Autonomous surveillance boats
- Propulsion health monitoring
- Predictive maintenance systems
- Industrial motor monitoring
- Fault-tolerant embedded control
The same architecture can be adapted to other rotating machinery.
🚀 Future Scope
The project can be extended with:
1. Real Sensors
Replace simulated inputs with calibrated:
- Current sensors
- Temperature sensors
- Hall-effect RPM sensors
- Industrial accelerometers
2. Advanced Vibration Analysis
Implement:
- RMS
- Peak
- Peak-to-peak
- Moving average
- FFT
- Frequency-domain analysis
3. Machine Learning
Machine-learning models could classify:
Normal
Propeller Damage
Bearing Fault
Motor Overheating
Propulsion Obstruction

4. Wireless Telemetry
Transmit:
RPM
Current
Temperature
Vibration
Health Score
Fault Status

to a remote ground station.
5. Fault History
Store fault information using:
- Flash
- EEPROM
- SD card
- External memory
6. Hardware-in-the-Loop
Connect the STM32 to actual propulsion hardware for controlled testing.
7. Physical Marine UAV
The final stage could integrate:
STM32
+
Motor
+
ESC
+
Propeller
+
Current Sensor
+
Temperature Sensor
+
Accelerometer
+
Battery

with appropriate safety procedures.
⚠️ Prototype Limitations
This project is currently a simulation/prototype.
The following are simulated:
- Motor current
- Motor temperature
- Propulsion RPM
- Propulsion actuator
The vibration algorithm is a lightweight dynamic acceleration-change indicator.
The system has not been certified or tested on an operational marine UAV.
Physical deployment would require:
- Calibrated sensors
- Hardware-in-the-loop testing
- Propulsion testing
- Safety validation
- Environmental testing
- Marine-system certification
📚 Documentation
Additional project documentation is available in the docs folder:
- Project Report – detailed technical documentation
- Flowchart – system operation and fault-control flow
- Validation Tests – simulation test cases and results
👨‍💻 Author
Sri Sujan VM
ECE | Embedded Systems | Robotics | AI & Automation
🔑 Keywords
STM32
NUCLEO-C031C6
Wokwi
Embedded Systems
Marine UAV
Autonomous Marine UAV
Propulsion Monitoring
Fault-Tolerant Control
Sensor Fusion
MPU6050
Vibration Monitoring
Predictive Maintenance
Real-Time Monitoring
Embedded AI
Industrial Automation
Motor Health Monitoring

⭐ Project Highlights
Real-time multi-sensor propulsion monitoring + fault classification + automatic fault-tolerant control on STM32.

The key feature of this project is that it does not only detect faults. It also changes the propulsion response based on fault severity, making it a foundation for intelligent and fault-tolerant autonomous propulsion systems.
