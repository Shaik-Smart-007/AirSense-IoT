# AirSense-IoT — Air Quality Monitoring System with Ventilation

A real-time indoor air quality, temperature, and humidity telemetry dashboard built for an **ESP32 DevKit V1** hardware prototype and **Firebase Realtime Database**, paired with a standalone single-file web console (`index.html`) and an offline **Demo Mode** for instant testing without hardware.

---

## Why AirSense-IoT Exists

Indoor smoke, cooking fumes, stagnant air, and rising gas levels can build up quietly before anyone notices. **AirSense-IoT** pairs environmental sensing and automated ventilation/alarm hardware on an ESP32 with a dark-mode telemetry web app that polls live JSON readings from Firebase—so you can monitor room conditions and hardware states from a browser while the prototype runs over a mobile hotspot.

---

## Key Features

- **0–3000 Air Quality Gauge:** Semi-circular SVG arc gauge scaled to `0–3000` with dynamic threshold tick positioning (`1500` default) and safe clamping if raw readings spike higher.
- **Automated Ventilation & Alert States:** Tracks `OK`, `WARMING_UP`, and `BAD AIR` states alongside **Exhaust Fan** (`GPIO 4`), **Active Buzzer** (`GPIO 14`), and **Green / Red Status LEDs** (`GPIO 26` / `GPIO 27`).
- **300-Second MQ-135 Preheat Tracker:** Built-in 5-minute (`300 s`) thermal stabilization progress bar with uptime sync, manual restart, and dismiss controls.
- **Live Trend Chart & Event Log:** Dual-axis real-time chart (last 40 readings) plotting MQ-135 and temperature against the threshold line, paired with a 20-entry transition log.
- **Offline Demo Mode & Smoke Spike Simulation:** Toggle **Demo mode** anytime to test live UI updates, alert states, a ~12-second simulated smoke spike, and a shortened 15-second demo fan cooldown without needing hardware or Wi-Fi.
- **Zero-Build Single-File Frontend:** Everything runs in a single `index.html` file with Chart.js loaded via CDN—no bundler, framework build step, or Gemini API key required.

---

## How Data Flows

```text
[DHT11 + MQ-135 Sensors]
          │
          ▼
  [ESP32 DevKit V1] ──(Mobile Hotspot Wi-Fi, HTTP PUT every ~2s)──▶ [Firebase Realtime Database]
                                                                            │
                                                                    (/sensor_data.json)
                                                                            │
                                                                   (HTTP GET every 2s)
                                                                            ▼
                                                              [index.html on Vercel]
1. The **ESP32 DevKit V1** reads environmental and gas sensor data, evaluates the air quality threshold (`1500`), and drives the fan, buzzer, and LEDs (including a ~2–3 minute post-smoke fan cooldown after air clears).
2. Over a **mobile hotspot**, the ESP32 sends JSON updates to `/sensor_data.json` in **Firebase Realtime Database**.
3. The web dashboard (`index.html`) polls `/sensor_data.json` every `2000 ms` (marking readings `STALE` if no fresh change arrives within `10000 ms`) and updates every card, gauge, chart, and log entry.

---

## Hardware Pin Map (ESP32 DevKit V1)

| Peripheral | ESP32 GPIO | Signal / Role |
| :--- | :--- | :--- |
| **DHT11 Temperature & Humidity** | `GPIO 18` | Digital single-wire environmental sensor |
| **MQ-135 Gas & Smoke Sensor** | `GPIO 34` | Analog ADC input (`0–3000` dashboard scale) |
| **Active Buzzer** | `GPIO 14` | Audible bad-air / smoke alarm output |
| **Green Status LED** | `GPIO 26` | Air OK visual indicator |
| **Red Status LED** | `GPIO 27` | Bad air / alert visual indicator |
| **Exhaust Fan Relay** | `GPIO 4` | Active ventilation relay control |

---

## Expected Firebase JSON Payload (`/sensor_data.json`)

```json
{
  "temperature": 27.4,
  "humidity": 54.0,
  "mq135": 620,
  "threshold": 1500,
  "status": "OK",
  "fanOn": false,
  "buzzerOn": false,
  "greenLedOn": true,
  "redLedOn": false,
  "uptimeSec": 420,
  "lastUpdated": 1760000000
}
