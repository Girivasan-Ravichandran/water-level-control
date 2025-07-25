
# ESP32 Water Level Control System (with PID & Web Server Integration)

This project uses an **ESP32**, **ultrasonic sensor**, and **PID control** to maintain the water level in a tank.
The system can:

* Measure water level using an **HC-SR04 ultrasonic sensor**.
* Automatically **control a valve** (via PWM/DAC) to maintain the desired water level.
* **Send live data to a web server** and **receive updated parameters (PID gains & setpoint)** over Wi-Fi.

---

## Features

* **Real-time water level monitoring** (percentage-based).
* **PID control loop** to keep the water level at the target (`setpoint`).
* **Web server communication**:

  * Sends water level updates via **HTTP POST**.
  * Retrieves PID gains (`Kp`, `Ki`, `Kd`) and `setpoint` via **HTTP GET**.
* **Flexible valve control**: Works with both **PWM** (for any GPIO) and **DAC** (on pins 25/26).
* Easily configurable tank height, Wi-Fi credentials, and server URL.

---

## Hardware Required

1. **ESP32 Development Board**
2. **Ultrasonic Sensor (HC-SR04 or JSN-SR04T)**
3. **Water Tank**
4. **Valve (solenoid/analog controlled)**
5. **12V/5V power supply (for valve)**
6. **Optional relay/MOSFET driver** (if using a solenoid valve)

---

## Circuit Connections

| ESP32 Pin  | Component               |
| ---------- | ----------------------- |
| GPIO 5     | Ultrasonic Trigger Pin  |
| GPIO 18    | Ultrasonic Echo Pin     |
| GPIO 25    | Valve Control (PWM/DAC) |
| 3.3V & GND | Sensor Power            |

---

## Software Setup

1. Install **Arduino IDE** and add **ESP32 board package**.

2. Install the required libraries:

   * `WiFi.h` (included in ESP32 package)
   * `HTTPClient.h` (included in ESP32 package)
   * `ArduinoJson` (install via Library Manager)

3. Update these in the code:

   ```cpp
   const char* ssid = "YOUR_WIFI_SSID";
   const char* password = "YOUR_WIFI_PASSWORD";
   const char* serverUrl = "http://your-web-server.com/api";
   const float tankHeight = 100.0; // Adjust to your tank height in cm
   ```

4. Flash the code to ESP32.

---

## How It Works

1. **Ultrasonic sensor** measures the distance to the water surface.
2. The code calculates **water level %** based on tank height.
3. The **PID controller** compares the water level with the `setpoint` and adjusts the valve opening (0–100%).
4. The ESP32 **sends the current water level** to a server every 5 seconds.
5. The ESP32 **fetches updated `Kp`, `Ki`, `Kd`, and setpoint** from the server, allowing remote tuning.

---

## Server API Specification

* **POST `/api/waterLevel`**
  Sends:

  ```json
  {
    "waterLevel": 45.3
  }
  ```
* **GET `/api/params`**
  Receives:

  ```json
  {
    "setpoint": 60,
    "kp": 2.5,
    "ki": 0.1,
    "kd": 0.5
  }
  ```

---

## Tuning the PID

* Start with `Kp` only (set `Ki` and `Kd` to `0`) to get basic response.
* Gradually add `Ki` to correct steady-state error.
* Add `Kd` to stabilize fast oscillations.
* You can adjust these values dynamically using the server.

---

## Serial Output Example

```
Water Level: 48.5%
PID: P=1.5, I=0.5, D=0.2, Control=55.2
Valve control: 55.2%
HTTP Response code: 200
Updated parameters - Setpoint: 60, Kp: 2.5, Ki: 0.1, Kd: 0.5
```

---

## Future Improvements

* Add **web dashboard** for real-time monitoring.
* Use **MQTT** instead of HTTP for faster data transfer.
* Implement **automatic tuning** of PID parameters.
* Add **failsafe (low-level cutoff)** for dry run protection.

