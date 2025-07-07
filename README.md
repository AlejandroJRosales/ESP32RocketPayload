# ESP32 Rocket Payload Architecture Overview
Architecture for ESP32 payload designed for a rocket where the payload is chosen for real-time data collection, telemetry transmission, and recovery after landing.

## System Overview

The payload integrates multiple sensors and communication modules to capture flight data and assist in recovery. The components are chosen for lightweight, low-power operation suitable for high-G, short-duration flights.

## Diagram
ESP32
├── GPS Module (UART)
├── LoRa Transceiver (SPI)
├── Barometric Pressure Sensor (I2C)
├── IMU: Accelerometer + Gyroscope + Magnetometer (I2C)
├── MicroSD Card Module (SPI or 2nd SPI bus)
├── Buzzer / Piezo Alarm (GPIO)
├── LED Beacon (GPIO with optional light sensor trigger)
└── Battery Monitor (Optional, I2C)

## Pin Configuration (Example)

| Peripheral           | Interface | Suggested ESP32 Pins   |
|----------------------|-----------|-------------------------|
| GPS (NEO-6M/BN-220)  | UART      | RX = GPIO16, TX = GPIO17|
| LoRa (RFM95/SX1276)  | SPI       | MISO = GPIO19, MOSI = GPIO23, SCK = GPIO18, NSS = GPIO5 |
| Barometer (BMP280)   | I2C       | SDA = GPIO21, SCL = GPIO22 |
| IMU (MPU6050/BNO055) | I2C       | SDA = GPIO21, SCL = GPIO22 |
| MicroSD Card         | SPI       | MISO = GPIO19, MOSI = GPIO23, SCK = GPIO18, CS = GPIO4 |
| Buzzer               | GPIO      | GPIO26 |
| LED Beacon           | GPIO      | GPIO27 |
| Battery Monitor      | I2C       | SDA = GPIO21, SCL = GPIO22 |

Note: Ensure non-conflicting use of SPI and I2C buses. Use `SPI.begin()` with custom pins if needed.

## Power Supply

- Battery: 3.7V LiPo, 500–1000 mAh recommended
- Voltage Regulation: Ensure 3.3V for ESP32 and peripherals
- Optional: Reed switch or toggle switch for safe launch pad arming

## Flight Sensors

| Sensor        | Data Collected         |
|---------------|------------------------|
| GPS           | Latitude, Longitude, Altitude, Speed |
| Barometer     | Altitude (high-resolution), Pressure |
| IMU           | Acceleration, Orientation, Spin rate |
| Magnetometer  | Heading, Attitude stabilization data |
| Temp Sensor   | Optional, for thermal profile |

## Telemetry and Recovery

- LoRa: Transmit live GPS + altitude + status packets to a ground station
- Buzzer: Activate after apogee or via timeout for acoustic location
- LED Beacon: Flashing light for night or low-light visibility
- Optional:
    - Bluetooth Beacon: Short-range fallback
    - WiFi AP Mode: Broadcast location over WiFi if within range
    - RF Tracker: Backup, long-range beacon with RF receiver

## Data Logging

- MicroSD stores flight logs including:
    - Timestamped sensor data
    - GPS coordinates and velocity
    - IMU and barometric data
    - Event markers (launch, apogee, landing)

## Design Considerations

- Secure components to withstand high-G forces (~10–15G)
- Isolate sensors with vibration-dampening material
- Antenna placement is critical for GPS and LoRa performance
- Thermal insulation may be needed for sensors in winter launches
- Weight budget must be managed carefully for the C6-5 rocket class

## Bill of Materials (BOM)

| Component         | Model / Type           |
|------------------|------------------------|
| MCU              | ESP32 Dev Module       |
| GPS Module       | u-blox NEO-6M / BN-220 |
| LoRa Transceiver | RFM95 / SX1276         |
| Barometer        | BMP280 / MS5611        |
| IMU              | MPU6050 / BNO055       |
| MicroSD Module   | Standard SPI interface |
| Buzzer           | 3.3V Piezo or active   |
| LED              | High-brightness LED    |
| Battery          | 3.7V LiPo 500–1000mAh  |
| Voltage Reg      | 3.3V LDO (if needed)   |

## Expansion Ideas

- Add altitude-triggered camera module
- Include environmental sensors (e.g., humidity, UV index)
- Use Kalman filtering for smoother altitude estimates
- Design a companion ground station with LoRa + display

## License

This project documentation is released under the MIT License.
