# ESP32 Rocket Payload Architecture Overview

This document outlines the design and architecture of an ESP32-based rocket payload system built for experimental model rocketry. Designed for rockets like the Estes Amazon (~30" tall, ~600+ ft apogee with a C6-5 engine), the payload supports real-time telemetry, in-flight sensor logging, onboard camera imaging, and post-flight recovery.

---

## Project Purpose and Goals

This project aims to create a compact, low-cost, and modular flight data system that can:

- **Log** high-frequency sensor and camera data to an onboard MicroSD card.
- **Transmit** telemetry in real-time during flight and descent via LoRa.
- **Assist in recovery** using audio (buzzer), visual (LED), and GPS location tools.
- **Experiment** across multiple rocket kits for performance testing, structural stress testing, and modular payload adaptability.

### Captured & Stored Data

- GPS coordinates, altitude, velocity  
- Barometric pressure and derived altitude  
- 3-axis acceleration, gyroscope, and magnetic field data  
- Temperature (from IMU or additional sensor)  
- Still images from onboard OV2640 camera  
- Event timestamps (e.g., launch, apogee, parachute deploy, landing)

### Transmitted Telemetry (via LoRa)

- Live GPS fix (lat/lon/alt)  
- Altitude (from barometer)  
- Basic IMU status  
- System state (e.g., in-flight, landed, recovery beacon active)

---

## System Architecture Diagram

    +--------------------------+
    |         ESP32            |
    |      (DevKit or CAM)     |
    +-----------+--------------+
                |
    +-----------+-------------------------------+
    |                                           |
    UART GPS                              SPI Bus (shared)
    |                                           |
    |                           +---------------+--------------+
    |                           |                              |
    |                    LoRa Transceiver            MicroSD Card Module
    |                                                         
    I2C Bus (shared)                             OV2640 Camera Module
    |                                                         |
    +----------+-----------------+              ESP32-CAM dedicated pins
               |                 |
     Barometer (e.g., BMP280)    |
     IMU (e.g., MPU6050)         |
     Magnetometer (optional)     |

    GPIO
      |
    +-----------+-----------+
    |                       |
    Buzzer              LED Beacon

    Optional: Battery Monitor via I2C (e.g., INA219)

---

## Pin Configuration (Example)

| Peripheral           | Interface | Suggested ESP32 Pins / Notes              |
|----------------------|-----------|-------------------------------------------|
| GPS Module           | UART      | RX = GPIO16, TX = GPIO17                  |
| LoRa Transceiver     | SPI       | MISO = GPIO19, MOSI = GPIO23, SCK = GPIO18, NSS = GPIO5 |
| Barometric Sensor    | I2C       | SDA = GPIO21, SCL = GPIO22                |
| IMU                  | I2C       | SDA = GPIO21, SCL = GPIO22                |
| MicroSD Card         | SPI       | MISO = GPIO19, MOSI = GPIO23, SCK = GPIO18, CS = GPIO4 |
| OV2640 Camera        | Parallel  | Requires ESP32-CAM board (dedicated pins) |
| Buzzer               | GPIO      | GPIO26                                    |
| LED Beacon           | GPIO      | GPIO27                                    |
| Battery Monitor      | I2C       | SDA = GPIO21, SCL = GPIO22 (optional)     |

> If using ESP32-CAM, pin availability is limited—plan around shared bus constraints.

---

## Power Supply

- Battery: 3.7V LiPo (500–1000 mAh recommended)  
- Voltage Regulation: Stable 3.3V needed for all peripherals  
- Launch Arming: Use reed switch or power switch for pad-side activation  
- Optionally monitor voltage using an INA219 or voltage divider + ADC

---

## Data Logging and Storage

The MicroSD card stores:

- High-frequency flight sensor data  
- Timestamps of flight phases  
- GPS coordinates at regular intervals  
- Camera images (OV2640)  
- System error/status logs  

Logging can be adjusted via a configuration file (optional feature) or baked into firmware constants.

---

## Telemetry and Recovery Aids

- **LoRa**: Sends GPS + altitude + status at intervals (customizable, e.g., 1 Hz)  
- **Buzzer**: Audio beacon activated by timer, ground detection, or altitude fall  
- **LED Beacon**: Bright flashes during descent or after landing  
- **WiFi (ESP32-CAM)**: Optional soft AP to serve landing coordinates via local web page  
- **Bluetooth (fallback)**: Optional for close-range recovery  

---

## Bill of Materials (BOM)

| Component         | Model / Type               |
|------------------|----------------------------|
| MCU              | ESP32 DevKit or ESP32-CAM  |
| GPS Module       | u-blox NEO-6M / BN-220     |
| LoRa Transceiver | RFM95 / SX1276             |
| Barometer        | BMP280 / MS5611            |
| IMU              | MPU6050 / BNO055           |
| Magnetometer     | HMC5883L (optional)        |
| MicroSD Module   | Built-in (ESP32-CAM) or SPI breakout |
| Camera Module    | OV2640 (via ESP32-CAM)     |
| Buzzer           | 3.3V active piezo          |
| LED              | High-intensity LED         |
| Battery          | 3.7V LiPo 500–1000 mAh     |
| Voltage Reg      | AMS1117 3.3V (if needed)   |

---

## Expansion Ideas

- Add Kalman filtering for better altitude/velocity fusion  
- Deploy a barometric parachute trigger or servo release  
- Design a modular payload system that swaps across rocket kits  
- Build a LoRa ground station with OLED, touchscreen, or mobile telemetry viewer  
- Stream camera images live (bandwidth-limited) with JPEG compression  

---

# Rocket Payload Thrust and Weight Estimation

## Step 1: Estimate Payload + Rocket Weight

Here’s a rough estimate of the weight of all major components:

| **Component**                 | **Estimated Weight (g)** |
|------------------------------|---------------------------|
| ESP32 microcontroller        | 8                         |
| GPS Module (e.g., BN-220)    | 10                        |
| LoRa Transceiver             | 5                         |
| Barometric Sensor (BMP280)   | 2                         |
| IMU (e.g., MPU6050)          | 3                         |
| MicroSD Module               | 2                         |
| Buzzer + LED                 | 5                         |
| OV2640 Camera (ESP32-CAM)    | 12                        |
| LiPo Battery (1S ~500mAh)    | 15–25                     |
| Wiring, headers, mount, shrink | 10–15                   |
| **Total Electronics**        | **~72–87 g**              |
| Airframe + Nose Cone + Fins  | ~45–55 g                  |
| **Estimated Total**          | **~120–140 g**            |

---

## Step 2: C6-5 Engine Thrust

- **Average thrust:** ~4.4 N  
- **Peak thrust:** ~12.1 N  
- **Burn time:** ~0.8 sec  
- **Total impulse:** ~8.8 Ns

---

## Step 3: Minimum Thrust Required

### To just lift off:

The force required to lift off is:

**F_required = m × g**  
**F_required = 0.14 kg × 9.81 m/s² = 1.3734 N**  
**F_required ≈ 1.37 N**

---

### To lift off safely (Thrust-to-Weight Ratio ≥ 5):

**F_required = 5 × m × g**  
**F_required = 5 × 0.14 kg × 9.81 m/s²**  
**F_required ≈ 6.87 N**

---

## Conclusion

The C6-5 engine’s **average thrust of 4.4 N** is **not enough** to reach the recommended thrust-to-weight ratio of 5, but it **exceeds the minimum thrust of 1.37 N**, so the rocket **will lift off**.

However:

- **Initial acceleration will be low**
- **Flight may be unstable**
- **Apogee may be lower**
- **Performance may degrade** if drag is high or the launch angle is not ideal

