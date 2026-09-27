# Real-Time-Energy-Monitoring-and-Data-Acquisition-System

# Project Overview

This project is designed to measure and display real-time electrical parameters—Voltage (V), Current (I), and Power (P)—using an STM32F446RE microcontroller. It also logs the data via UART for further analysis or integration into data acquisition systems.

The system is suitable for applications like:

- Home energy monitoring
- Industrial load analysis
- Smart meters
- Real-time IoT energy dashboards

# Components Used
````
| Component       | Model / Type                 | Function                                                   |
| --------------- | ---------------------------- | ---------------------------------------------------------- |
| Microcontroller | STM32F446RE                  | Core processor for ADC, DMA, UART, GPIO control            |
| Current Sensor  | ACS712 (5A / 20A / 30A)      | Measures current flowing through the load                  |
| Voltage Sensor  | Voltage divider module       | Measures voltage across the load                           |
| LCD             | 16x2 Character LCD           | Displays voltage, current, and power readings in real-time |
| DMA             | STM32 DMA2_Stream0           | Transfers ADC readings to memory without CPU intervention  |
| UART            | USART2                       | Sends real-time sensor data to PC or logging device        |
| Miscellaneous   | Resistors, wires, breadboard | Connections and interfacing                                |
`````


