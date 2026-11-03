💊 MediTrack – Smart Pill Reminder Firmware & Linux Host
<div align="center">
https://img.shields.io/badge/C%2B%2B-00599C?style=for-the-badge&logo=c%2B%2B&logoColor=white
https://img.shields.io/badge/ESP32-E7352C?style=for-the-badge&logo=espressif&logoColor=white
https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white
https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black
https://img.shields.io/badge/Pytest-0A9EDC?style=for-the-badge&logo=pytest&logoColor=white

A robust, fault-tolerant IoT medication management system featuring a multithreaded Linux host and resilient ESP32 firmware.

</div>
📖 Overview
MediTrack is an end-to-end embedded system designed to ensure patients never miss a dose. It combines an ESP32-based hardware dispenser with a real-time Linux desktop dashboard. The system is built for reliability, featuring a fault state machine, debounced GPIO, power-loss-safe circular logging, and a CRC-protected UART communication protocol.

✨ Key Highlights
Resilient Firmware: RTC-interrupt dose scheduler, debounced GPIO, and a fault state machine.

Power-Loss Safe Logging: CRC-16 circular buffer that survives unexpected power cuts mid-write.

Robust Host Application: Multithreaded Python/Linux host using a CRC-8 UART protocol.

Advanced Concurrency: Bounded queue with backpressure and explicit drop accounting.

100% Tested: 24 comprehensive pytest regression tests with documented root-cause analyses for failure injection.

📸 Project Showcase
🖥️ Linux Host Dashboard
The Python-based dashboard provides a clean UI to monitor schedules, adherence rates, device status, and live system logs.

<p align="center"> <img src="assets/final%20dashborad.png" alt="MediTrack Expanded Dashboard" width="100%"> </p> <p align="center"> <img src="assets/sample%20dash.png" alt="MediTrack Compact Dashboard" width="100%"> </p>
🔌 Hardware Prototype
A physical 7-day pill dispenser equipped with an ESP32, RTC, LCD screen, physical navigation buttons, and per-day LED indicators.

<p align="center"> <img src="assets/meditrack%20pillbox.png" alt="MediTrack Hardware Prototype" width="80%"> </p>
⚙️ System Architecture & Features
1. ESP32 Firmware (C/C++)
RTC-Interrupt Scheduler: Precise dose timing using a hardware Real-Time Clock.

Fault State Machine: Gracefully handles system states (INIT, READY, ALARM, ACKED, FAULT).

Debounced GPIO: Hardware-level button filtering to prevent false acknowledgments.

Data Integrity: Uses Non-Volatile Storage (NVS) as EEPROM with a CRC-16 protected circular buffer.

2. Linux Host (Python)
Multithreaded Architecture: Dedicated threads for serial reading and writing to prevent blocking.

Custom Protocol: Implements a lightweight 0xAA | len | type | payload | CRC-8 frame structure.

Reliability: Implements ACK/retry mechanisms for lost packets.

Backpressure Management: Uses a bounded queue (queue.Queue(maxsize=100)) to prevent memory overflow, with robust drop accounting.

🛠️ Engineering & Root-Cause Analysis
During development, four critical system failures were identified, root-caused (using serial logs and a logic analyzer), and fixed:

Button Bounce: Fixed by implementing a 30ms software debounce.

Power-Loss Corruption: Fixed by writing the CRC-16 record last and validating on boot.

Alarm Missed During Blocking Flash Write: Fixed by decoupling the RTC interrupt from the flash write routine.

Queue Starvation: Fixed by implementing a bounded queue with backpressure drop policies.

🧪 Automated Testing & Validation
Robustness is validated via a comprehensive regression suite. The system runs entirely automated tests for protocol encoding/decoding, CRC validation, ACK/retry logic, queue limits, and log parsing.

<div align="center"> <img src="assets/serial%20log.png" alt="Serial Log UART Communication" width="48%"> <img src="assets/reg%20tests.png" alt="Pytest Regression Tests" width="48%"> </div>
text
$ pytest -q
====================== test session starts ======================
collected 24 items
test_scheduler.py .........                                [ 37%]
test_uart_protocol.py ......                               [ 62%]
test_fault_recovery.py .......                             [ 91%]
test_queue.py ..                                           [100%]
====================== 24 passed in 6.21s ======================
🚀 Getting Started
Prerequisites
Hardware: ESP32 Dev Board, DS3231 RTC, Pushbuttons, LEDs, Buzzer.

Software: PlatformIO (for ESP32), Python 3.9+ (for Linux Host).

Installation & Execution
Flash the Firmware:
Open the firmware/ directory in PlatformIO, connect the ESP32 via USB, and upload.

Run the Host:

bash
cd host/
pip install -r requirements.txt
python host.py --port /dev/ttyUSB0
Run Tests:

bash
pytest -v
