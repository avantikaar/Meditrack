# 💊 MediTrack
Smart Pill Reminder Firmware & Linux Host

[![C/C++](https://img.shields.io/badge/C%2B%2B-00599C?style=flat-square&logo=c%2B%2B&logoColor=white)](https://isocpp.org/)
[![ESP32](https://img.shields.io/badge/ESP32-E7352C?style=flat-square&logo=espressif&logoColor=white)](https://www.espressif.com/)
[![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black)](https://www.linux.org/)
[![Pytest](https://img.shields.io/badge/Pytest-0A9EDC?style=flat-square&logo=pytest&logoColor=white)](https://docs.pytest.org/)

---

## 📸 Screenshots

### Main Dashboard
![Final Dashboard](./final%20dashborad.png)

### Compact Dashboard View
![Sample Dashboard](./sample%20dash.png)

### Hardware Prototype
![MediTrack Hardware](./meditrack%20pillbox.png)

### UART Serial Log (CRC-8)
![Serial Log](./serial%20log.png)

### Pytest Regression Suite
![Regression Tests](./reg%20tests.png)

---

## 📌 Overview

**MediTrack** is an end-to-end embedded system designed to ensure patients never miss a dose. It combines an ESP32-based physical dispenser with a multithreaded Linux desktop dashboard. The system is built for reliability, featuring a fault state machine, debounced GPIO, power-loss-safe circular logging, and a CRC-protected UART communication protocol.

The project addresses a critical healthcare problem: medication non-adherence. By providing hardware-level alarms and a robust logging system, MediTrack ensures that doses are tracked accurately, even in the event of unexpected power loss or system faults.

The platform:
*   Schedules doses using a hardware **RTC (Real-Time Clock)**.
*   Handles physical button interactions with **debounced GPIO**.
*   Survives power-loss events mid-write using a **CRC-16 circular buffer**.
*   Communicates with a Linux host via a **CRC-8 UART protocol** with ACK/retry.

---

## ✨ Features

| Feature | Description |
| :--- | :--- |
| ⏰ **RTC-Interrupt Scheduler** | Precise dose timing using a hardware Real-Time Clock to ensure alarms are never missed. |
| 🛡️ **Power-Loss Safe Logging** | Implements a CRC-16 protected circular buffer in NVS/EEPROM that recovers safely from unexpected brownouts. |
| 🔄 **Fault State Machine** | Gracefully handles system states (`INIT`, `READY`, `ALARM`, `ACKED`, `FAULT`) with robust error recovery. |
| 🐍 **Multithreaded Linux Host** | Python-based host application with dedicated reader/writer threads, preventing UI blocking. |
| 📊 **Backpressure & Queueing** | Bounded queue (`queue.Queue`) prevents memory overflow, with explicit drop accounting during high traffic. |
| 🧪 **Extensive Test Suite** | 24 automated `pytest` regression tests covering protocol encoding, ACK/retry logic, and fault injection. |

### The Hardware-to-Host Pipeline
Every alarm and user interaction flows through a structured, fault-tolerant pipeline:

| Stage | Component | Responsibility |
| :--- | :--- | :--- |
| **1** | **ESP32 Firmware** | RTC triggers alarm, GPIO debounces button press, state machine updates. |
| **2** | **Data Integrity** | Event is written to the CRC-16 circular buffer (survives power loss). |
| **3** | **UART Protocol** | Event is framed with a CRC-8 checksum and transmitted via serial. |
| **4** | **Python Host** | Dedicated thread receives data, validates CRC, enqueues to bounded queue, and updates the UI. |

*Production deployment note: The host application is designed to simulate power-loss and fault injection during testing to ensure the CRC-16 recovery logic works flawlessly before hardware deployment.*

---

## 🛠️ Tech Stack

**Firmware (ESP32)**
*   C/C++ (Arduino Framework / ESP-IDF)
*   RTC & NVS (Non-Volatile Storage)
*   Hardware Interrupts & GPIO Polling

**Linux Host (Python)**
*   Python 3.9+
*   Multithreading (`threading`, `queue.Queue`)
*   PySerial (UART Communication)
*   Tkinter / PyQt (Dashboard UI)

**Testing & Debugging**
*   Pytest (24 Regression Tests)
*   Logic Analyzer (Hardware debugging)
*   Serial Logs (Fault injection root-cause analysis)

---

## 🔌 UART Protocol Reference

The system uses a custom lightweight frame structure: `0xAA | len | type | payload | CRC-8`.

| Direction | Command | Description | CRC |
| :--- | :--- | :--- | :--- |
| ESP32 → Host | `DOSE_ALERT` | Triggered when RTC alarm fires | ✅ CRC-8 |
| ESP32 → Host | `DOSE_ACK` | Triggered when user presses button | ✅ CRC-8 |
| Host → ESP32 | `ACK` | Acknowledges receipt of an alert | ✅ CRC-8 |
| Host → ESP32 | `SET_SCHEDULE` | Updates the RTC schedule | ✅ CRC-8 |

---

## 🧪 Testing

The system includes a comprehensive test suite to validate protocol integrity, queue management, and fault recovery.

```text
$ pytest -q
====================== test session starts ======================
collected 24 items
test_scheduler.py .........                                [ 37%]
test_uart_protocol.py ......                               [ 62%]
test_fault_recovery.py .......                             [ 91%]
test_queue.py ..                                           [100%]
====================== 24 passed in 6.21s ======================
