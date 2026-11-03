# MediTrack — Smart Pill Reminder Firmware & Linux Host

MediTrack is a small embedded-systems project that demonstrates dose reminders, event acknowledgement, CRC-protected UART frames, persistent event journaling, and a Linux/Python host dashboard.

> **Important:** This repository contains a working browser dashboard in demo mode, a Python host/protocol implementation, a starter ESP32 firmware sketch, and automated tests. The browser demo does not communicate with physical hardware until the serial integration is configured and tested.

## Features

- Interactive dashboard: daily doses, medication list, dose acknowledgement, device status, event history and CSV export.
- Python host: CRC-8 UART framing, ACK response, bounded event queue and drop accounting.
- Persistent host journal: CRC-16 protected JSONL records; invalid records are skipped during recovery.
- ESP32 firmware starter: scheduled reminder, button acknowledgement, LED/buzzer indication and Preferences-backed event sequence.
- Pytest suite for CRC, frame encoding/decoding, journal recovery and queue behaviour.

## Repository structure

```text
MediTrack/
├── dashboard/             # Static interactive web dashboard
│   ├── index.html
│   ├── styles.css
│   └── app.js
├── firmware/
│   └── MediTrack.ino      # ESP32 Arduino starter firmware
├── host/
│   └── meditrack_host.py  # Python host, UART protocol and journal
├── tests/
│   └── test_meditrack.py
├── requirements.txt
├── .gitignore
└── README.md
```

## Run the dashboard

Open `dashboard/index.html` directly in a browser. No build step or npm install is required.

The dashboard includes demo interactions:
1. Add a medication reminder.
2. Mark a dose as taken.
3. Simulate device disconnect/reconnect.
4. View event history.
5. Export event logs as CSV.
6. Run browser-side demo checks.

## Run the Python host in demo mode

Python 3.10+ recommended.

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
# macOS/Linux:
source .venv/bin/activate

pip install -r requirements.txt
python host/meditrack_host.py --demo
```

Demo mode generates sample reminder and acknowledgement events, stores them in `meditrack_events.jsonl`, and prints status.

## Connect a serial device

Install `pyserial` (included in requirements), then:

```bash
python host/meditrack_host.py --port COM5 --baud 115200
# Linux example:
python host/meditrack_host.py --port /dev/ttyUSB0 --baud 115200
```

The host expects MediTrack frames in this format:

```text
0xAA | LEN | TYPE | PAYLOAD... | CRC8
```

CRC-8 uses polynomial `0x07`, initial value `0x00`, over `LEN + TYPE + PAYLOAD`.
The host sends a one-byte `0x06` ACK after a valid frame.

## ESP32 firmware

1. Install Arduino IDE or PlatformIO.
2. Install the **RTClib** library by Adafruit.
3. Select an ESP32 board and open `firmware/MediTrack.ino`.
4. Check the pin definitions at the top of the sketch before wiring.
5. Upload and open Serial Monitor at 115200 baud.

Default pins in the sketch:
- LED: GPIO 2
- Buzzer: GPIO 25
- Acknowledge button: GPIO 27 (button to GND; internal pull-up)
- DS3231: I2C default pins for your ESP32 board

The firmware sketch is a starter implementation. Verify your board's pinout and RTC wiring before powering the circuit. Do not use real medication or rely on this prototype for medical dosing decisions.

## Run tests

```bash
pytest -q
```

## Protocol notes

- CRC-8 protects UART frames against transmission errors.
- CRC-16/CCITT-FALSE protects each host journal record.
- The queue is bounded (default 100 entries); if full, new events are counted as dropped instead of consuming unbounded memory.
- The local journal is an engineering demo. For production power-loss guarantees, use a carefully designed append-only flash journal with atomic commit markers and test it on the target hardware.

## Current implementation boundary

The dashboard is fully interactive with local demo data. The Python host and firmware are separate components that can be connected over USB serial after hardware testing. The dashboard-to-host live bridge is intentionally left as a next integration step rather than pretending that browser demo data is coming from the ESP32.
