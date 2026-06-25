# STM32 GPS Data Reception and Processing using NEO-6M

## Overview

This project implements GPS data reception and processing using an **STM32 microcontroller** and a **NEO-6M GPS module**.

The GPS module continuously transmits NMEA sentences through a UART interface. To ensure efficient and non-blocking communication, the STM32 receives GPS data using **UART with DMA (Direct Memory Access)** combined with **interrupts**. This approach allows continuous acquisition of GPS frames while minimizing CPU workload.

The received NMEA frames are then parsed to extract useful navigation information such as:

* Latitude
* Longitude
* UTC Time
* Date
* GPS Fix Status
* Number of Satellites

---

## Features

* UART communication with NEO-6M GPS module
* Continuous GPS data acquisition using DMA
* Interrupt-driven data processing
* NMEA sentence parsing
* Extraction of geographic coordinates and time information
* Low CPU utilization thanks to DMA
* Real-time GPS data handling

---

## Hardware Components

### Microcontroller

* STM32 Microcontroller (STM32F4xx / STM32F1xx / adapt to your board)

### GPS Module

* u-blox NEO-6M GPS Module

### Additional Components

* USB-to-UART (for debugging)
* Breadboard and jumper wires
* Power supply (3.3V / 5V depending on board)

---

## System Architecture

```text
+------------------+
|   NEO-6M GPS     |
|                  |
|  TX -----> RX    |
+--------+---------+
         |
         | UART
         v
+------------------+
|      STM32       |
|                  |
| UART + DMA RX    |
|                  |
| NMEA Parser      |
|                  |
| Data Processing  |
+--------+---------+
         |
         |
         v
+------------------+
| Debug Terminal   |
|   (UART/USB)     |
+------------------+
```

---

## Communication Flow

1. The NEO-6M module transmits NMEA sentences periodically.
2. STM32 UART receives incoming bytes.
3. DMA stores received data directly into memory.
4. DMA generates an interrupt when the reception buffer is full or when an event occurs.
5. The interrupt callback notifies the application.
6. NMEA frames are parsed.
7. Extracted GPS information becomes available for further processing or display.

---

## UART Configuration

| Parameter | Value    |
| --------- | -------- |
| Baud Rate | 9600 bps |
| Data Bits | 8        |
| Stop Bits | 1        |
| Parity    | None     |
| Mode      | RX       |
| DMA       | Enabled  |

---

## DMA Reception with Interrupts

The project uses DMA in circular mode to continuously receive GPS data.

### Advantages

* Non-blocking reception
* Reduced CPU load
* Continuous acquisition of GPS frames
* Improved system responsiveness

### DMA Workflow

```text
GPS Data
    |
    v
UART RX
    |
    v
DMA Transfer
    |
    v
Memory Buffer
    |
    v
DMA Interrupt
    |
    v
NMEA Processing
```

### Example Initialization

```c
HAL_UART_Receive_DMA(&huart1, gpsBuffer, GPS_BUFFER_SIZE);
```

### DMA Callback

```c
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
    if (huart->Instance == USART1)
    {
        gpsDataReady = 1;
    }
}
```

---

## NMEA Sentences

The NEO-6M outputs standard NMEA messages.

### Example GPGGA Frame

```text
$GPGGA,123519,4807.038,N,01131.000,E,1,08,0.9,545.4,M,46.9,M,,*47
```

### Example GPRMC Frame

```text
$GPRMC,123519,A,4807.038,N,01131.000,E,022.4,084.4,230394,003.1,W*6A
```

---

## NMEA Parsing

The parser identifies relevant NMEA sentences and extracts useful fields.

### Extracted Parameters

| Parameter        | Source        |
| ---------------- | ------------- |
| Latitude         | GPGGA / GPRMC |
| Longitude        | GPGGA / GPRMC |
| UTC Time         | GPGGA / GPRMC |
| Date             | GPRMC         |
| GPS Fix Status   | GPGGA         |
| Satellites Count | GPGGA         |

### Parsing Example

```c
char *token;

token = strtok(nmeaSentence, ",");

while(token != NULL)
{
    // Process fields
    token = strtok(NULL, ",");
}
```

---

## Project Structure

```text
├── Core
│   ├── Inc
│   │   ├── gps.h
│   │   └── main.h
│   └── Src
│       ├── gps.c
│       ├── main.c
│       └── stm32xx_it.c
│
├── Drivers
│
├── README.md
│
└── STM32CubeIDE Project Files
```

---

## Results

The system successfully receives and processes GPS data in real time.

### Example Output

```text
UTC Time     : 14:32:18
Latitude     : 36.7525 N
Longitude    : 3.0420 E
Satellites   : 10
Fix Status   : Valid
Date         : 12/06/2025
```

### Performance Observations

* Reliable GPS frame reception
* No data loss during continuous operation
* Low processor utilization thanks to DMA
* Stable UART communication at 9600 baud

---

## Future Improvements

* Display coordinates on an LCD or OLED screen
* Integrate SD card logging
* Add FreeRTOS task management
* Implement GPS speed and distance calculations
* Transmit data via Bluetooth or LoRa
* Visualize position on a map application

---

## Technologies Used

* STM32CubeIDE
* STM32 HAL Drivers
* UART Communication
* DMA
* Interrupt Handling
* NMEA Protocol
* Embedded C

---

## Author

Developed as an embedded systems project for GPS data acquisition and processing using STM32 and the NEO-6M GPS module.
