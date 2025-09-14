# IoT-based Robotic Car — OWN-SERVER

This repository contains the C++ firmware and server-integration code for an IoT-enabled robotic car controlled via your own server (OWN-SERVER). The project provides a lightweight, extensible C++ codebase to run on microcontrollers (ESP32 / ESP8266 / similar) or an on-device controller and connect to a self-hosted control server using HTTP or MQTT.

Key repo language: C++ (100%).

---

## Features

- Connects to a user-owned server (HTTP REST or MQTT) for remote control
- Drive control (forward/reverse/turn), speed control, and motor/servo control
- Sensor hooks (ultrasonic, IR, line sensors) for obstacle detection and autonomous behavior
- OTA-friendly layout for integration with PlatformIO/Arduino workflows
- Simple JSON-based command format for interoperability with dashboards and mobile apps

---

## Hardware (suggested)

- Microcontroller: ESP32 (recommended) or ESP8266 / Arduino with network module
- Motor driver: L298N, TB6612, or similar (appropriate for motor voltage/current)
- DC motors + wheels, caster wheel
- Power: Separate battery pack for motors and stable supply for the microcontroller
- Optional sensors: HC-SR04 (ultrasonic), IR sensors, line-tracking sensors, MPU6050 (IMU)
- Optional servos for camera tilt/grip

Always use a common ground between motor power and microcontroller power.

---

## Typical Wiring (example)

- Motor driver IN1/IN2 -> MCU GPIO (left motor)
- Motor driver IN3/IN4 -> MCU GPIO (right motor)
- Motor Vcc -> Battery pack (do not power motors from MCU 5V regulator)
- Motor GND -> Battery GND (shared with MCU GND)
- Ultrasonic: TRIG -> GPIO, ECHO -> GPIO (use level shifting for 5V signals if necessary)
- I2C sensors -> SDA/SCL

Adjust pins to match the configuration header in the firmware.

---

## Software requirements

- Arduino IDE or PlatformIO for building firmware
- If using ESP boards: add ESP32 or ESP8266 board support
- Common libraries (install via Library Manager or PlatformIO):
  - WiFi.h / ESP8266WiFi.h
  - PubSubClient (MQTT) — optional
  - ArduinoJson — optional (for JSON parsing)
  - Servo / Motor control libraries (if used)
  - Wire (I2C)

---

## Configuration

Edit the configuration constants at the top of the main firmware file (or in a dedicated config header) to set:

- WIFI_SSID, WIFI_PASSWORD
- SERVER_HOST or MQTT_BROKER (IP or hostname)
- SERVER_PORT (e.g., 80, 8080, 1883)
- DEVICE_ID or CLIENT_ID
- Pin mappings: LEFT_MOTOR_PWM, RIGHT_MOTOR_PWM, DIR pins, sensor pins
- Movement calibration, max speed, PWM frequency

Do not commit sensitive credentials to a public repository. Use placeholders or external configuration.

---

## Build & Upload

Arduino IDE:
1. Clone the repo:
   ```bash
   git clone https://github.com/yasinarafathemon/IoT-based-robotic_carOWN-SERVER.git
   ```
2. Open the main sketch (e.g., `firmware/robotic_car.ino`) in Arduino IDE.
3. Select the correct board and port, install any missing libraries, edit config, upload.

PlatformIO (VS Code):
1. Open repository in VS Code with PlatformIO extension.
2. Set the environment in `platformio.ini` to your board (esp32dev, nodemcuv2, etc.).
3. Add lib_deps if needed, edit config, then build/upload with PlatformIO.

---

## Network API Examples

HTTP command (POST JSON to /control):
```json
POST /control
Content-Type: application/json
{
  "device": "car-01",
  "action": "drive",
  "direction": "forward",
  "speed": 120
}
```

MQTT command (topic: devices/car-01/commands):
```json
{
  "action": "turn",
  "direction": "left",
  "angle": 40
}
```

Implement authentication or token-based access on your OWN-SERVER to prevent unauthorized control.

---

## Usage

1. Configure credentials and server details in the firmware.
2. Upload firmware to the board and power the motors with appropriate supply.
3. Monitor Serial output for network connection status and debug logs.
4. Send control messages from your server, dashboard, or MQTT client.
5. Add safety checks (stop on obstacle) to avoid collisions.

---

## Directory structure (expected / suggested)
- firmware/               — main C++ source files and sketches
- examples/               — example control clients and payloads
- server/                 — example OWN-SERVER code (optional)
- hardware/               — wiring diagrams, BOM, schematics
- docs/                   — setup guides and notes
- README.md               — this file
- LICENSE                 — license file (MIT recommended)

Adjust to match the actual layout in this repository.

---

## Troubleshooting

- Motors cause MCU resets: ensure separate power supply for motors and common ground.
- Wi‑Fi not connecting: verify SSID/password and antenna/board settings.
- MQTT/HTTP not reachable: check firewall, server port, and network reachability.
- Unreliable sensor readings: add filtering and ensure correct wiring and voltage levels.

---

## Contributing

Contributions are welcome. Typical workflow:
1. Fork the repo.
2. Create a feature branch.
3. Add changes and test on hardware/emulator.
4. Submit a pull request with a clear description and hardware used.

Include wiring diagrams and exact board/firmware versions for reproducibility.

---

## License

Suggested: MIT License. Add a LICENSE file if not present.

---

## Author

Yasin Arafat Emon  
GitHub: https://github.com/yasinarafathemon

---

Notes
- This README is written to be generic and action-oriented; please update pin names, filenames, and example paths to match the actual files in this repository.
