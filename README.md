# Real-Time Energy Monitoring and Data Acquisition System

A microcontroller-based **Real-Time Energy Monitoring and Data Acquisition System** developed using the **STM32F446RE**.

The system acquires voltage and current sensor signals through the STM32 ADC, transfers ADC data using **DMA**, calculates electrical parameters such as voltage, current, and power, displays the readings on a **16×2 LCD**, and sends the measurements through **UART** for real-time monitoring and data logging.

---

## 📌 Project Overview

Monitoring electrical parameters in real time is important in applications such as energy monitoring, load analysis, equipment monitoring, and data acquisition systems.

In this project, the STM32F446RE acts as the central controller.

The basic operation is:

```text
Voltage Sensor ──┐
                 │
                 ▼
              STM32 ADC
                 │
Current Sensor ──┘
                 │
                 ▼
                DMA
                 │
                 ▼
          Data Processing
                 │
        ┌────────┴────────┐
        ▼                 ▼
      16×2 LCD           UART
        │                 │
        ▼                 ▼
  Local Display       PC / Logger
```

The project demonstrates practical implementation of:

* ADC interfacing
* DMA-based data acquisition
* Sensor data conversion
* Real-time calculations
* LCD interfacing
* UART communication
* GPIO-based alert indication
* Embedded C programming using STM32 HAL

---

## ✨ Features

* Real-time voltage measurement
* Real-time current measurement
* Power calculation
* 12-bit ADC acquisition
* DMA-based ADC data transfer
* 16×2 LCD display
* UART data logging at 115200 baud
* Over-voltage / over-current alert indication
* STM32 HAL-based firmware
* Modular peripheral initialization using STM32CubeIDE

---

## 🛠️ Hardware Components

| Component         | Model / Type    | Purpose                             |
| ----------------- | --------------- | ----------------------------------- |
| Microcontroller   | STM32F446RE     | Main processing and control         |
| Current Sensor    | ACS712          | Current measurement                 |
| Voltage Sensor    | Voltage Divider | Voltage measurement                 |
| Display           | 16×2 LCD        | Local parameter display             |
| USB/UART          | USART2          | Data logging and monitoring         |
| LED               | GPIO controlled | Alert indication                    |
| Breadboard/PCB    | —               | Hardware connections                |
| Resistors & Wires | —               | Signal conditioning and connections |

---

## 🔌 Pin Configuration

### Voltage and Current Sensors

| Signal                | STM32 Pin | Peripheral     |
| --------------------- | --------- | -------------- |
| Voltage Sensor Output | PA0       | ADC1 Channel 0 |
| Current Sensor Output | PA1       | ADC1 Channel 1 |

### UART

| UART Signal | STM32 Pin |
| ----------- | --------- |
| TX          | PA2       |
| RX          | PA3       |
| Baud Rate   | 115200    |

### Alert LED

| Function  | STM32 Pin |
| --------- | --------- |
| Alert LED | PA5       |

### LCD

The 16×2 LCD is interfaced in **8-bit mode** using GPIO pins.

| LCD Signal | STM32 Pin               |
| ---------- | ----------------------- |
| RS         | PA7                     |
| EN         | PA6                     |
| Data lines | PA8, PA9, PA10, PB3–PB6 |

> Verify the final LCD pin mapping against the actual STM32CubeIDE project before connecting hardware.

---

# ⚙️ Working Principle

## 1. Sensor Measurement

The system uses two analog sensors:

### Voltage Measurement

A voltage divider scales the input voltage to a level suitable for the STM32 ADC.

The basic voltage-divider relationship is:

```text
Vout = Vin × R2 / (R1 + R2)
```

The STM32 measures `Vout` and applies the required scaling factor to estimate the original voltage.

### Current Measurement

The **ACS712** uses the Hall-effect principle to measure current.

Its output voltage changes according to the current flowing through the sensor.

For the example calibration used in the firmware:

```text
Current = (Sensor Voltage - 2.5) × 10
```

The exact sensitivity depends on the ACS712 version being used.

---

# 🔢 2. ADC Conversion

The STM32F446RE ADC is configured for **12-bit resolution**.

Therefore, the ADC produces values from:

```text
0 → 4095
```

The ADC value is converted into voltage using:

```c
v_raw = (adc_value × 3.3) / 4095;
```

Similarly, the current sensor ADC reading is converted into its corresponding sensor voltage.

---

# 🚀 3. DMA-Based Data Acquisition

DMA is used to automatically transfer ADC conversion results into memory.

The ADC values are stored in:

```c
uint16_t adc_buffer[CHANNELS];
```

with:

```c
#define CHANNELS 2
```

The two channels correspond to:

```text
adc_buffer[0] → Voltage
adc_buffer[1] → Current
```

Instead of requiring the CPU to manually read every ADC conversion, DMA transfers the data automatically.

### Why DMA is useful

```text
ADC
 │
 │ Conversion complete
 ▼
DMA
 │
 │ Automatic memory transfer
 ▼
adc_buffer[]
 │
 ▼
CPU processes data
```

This reduces CPU involvement in the data-transfer process and is useful in continuous data-acquisition applications.

---

# 🧮 4. Voltage, Current and Power Calculation

After the ADC data is available, the firmware converts the raw values into electrical parameters.

### Voltage

```c
float v_raw = (adc_buffer[0] * 3.3f) / 4095.0f;
voltage = v_raw * 5.0f;
```

The factor `5.0` represents the voltage-scaling calibration used in the example firmware.

### Current

```c
float i_raw = (adc_buffer[1] * 3.3f) / 4095.0f;
current = (i_raw - 2.5f) * 10.0f;
```

### Power

```c
power = voltage * current;
```

Therefore:

```text
P = V × I
```

where:

* `P` = Power
* `V` = Voltage
* `I` = Current

> The displayed values depend on the sensor characteristics and calibration constants used in the firmware.

---

# 🖥️ 5. LCD Display

The measured values are displayed on a 16×2 LCD.

Example output:

```text
V:5.0V I:0.5A
P:2.50 Watt
```

The LCD is interfaced using GPIO in **8-bit mode**.

The firmware contains helper functions for:

```c
lcd_initialise()
lcd_command()
lcd_data()
lcd_string()
Printdata()
```

These functions handle LCD initialization, commands, characters, and strings.

---

# 📡 6. UART Data Logging

The system sends measurement data to a PC or serial terminal using **USART2**.

Configuration:

```text
Baud Rate : 115200
Data Bits : 8
Parity    : None
Stop Bits : 1
```

Example UART output:

```text
LOG: V=5.00, I=0.50, P=2.50
```

This allows the measured data to be viewed or captured externally for further analysis.

---

# 🚨 7. Alert System

A GPIO-controlled LED is used to indicate an abnormal measurement condition.

The example firmware checks:

```c
if(voltage > 10 || current > 2)
```

If either condition is satisfied, the alert LED is switched ON.

Otherwise, it remains OFF.

```text
Normal Condition
       │
       ▼
 LED OFF

Voltage > Threshold
       OR
Current > Threshold
       │
       ▼
 LED ON
```

The threshold values are configurable according to the application.

---

# 💻 Software and Development Tools

* **STM32CubeIDE**
* **Embedded C**
* **STM32 HAL**
* **STM32CubeMX configuration**
* Serial terminal / UART monitor

---

# 📚 Important STM32 HAL Functions

| Function                     | Purpose                            |
| ---------------------------- | ---------------------------------- |
| `HAL_ADC_Init()`             | Initializes ADC                    |
| `HAL_ADC_Start_DMA()`        | Starts ADC conversion with DMA     |
| `HAL_ADC_ConvCpltCallback()` | Handles completed ADC DMA transfer |
| `HAL_UART_Transmit()`        | Sends data through UART            |
| `HAL_GPIO_WritePin()`        | Controls GPIO output               |
| `HAL_Delay()`                | Provides delay                     |
| `SystemClock_Config()`       | Configures STM32 system clock      |

---

# 🔄 Firmware Workflow

```text
                 START
                   │
                   ▼
          System Clock Setup
                   │
                   ▼
          GPIO / LCD Setup
                   │
                   ▼
             ADC1 Setup
                   │
                   ▼
              DMA Setup
                   │
                   ▼
             UART2 Setup
                   │
                   ▼
        Start ADC + DMA
                   │
                   ▼
       ADC Conversion Complete
                   │
                   ▼
          DMA → adc_buffer
                   │
                   ▼
        Convert ADC Readings
                   │
             ┌─────┴─────┐
             ▼           ▼
          Voltage      Current
             │           │
             └─────┬─────┘
                   ▼
             Power = V × I
                   │
          ┌────────┴────────┐
          ▼                 ▼
       Update LCD        Send UART
          │                 │
          └────────┬────────┘
                   ▼
             Check Alert
                   │
                   ▼
               Repeat
```

---

# 🧩 Code Structure

A typical project structure is:

```text
Real-Time-Energy-Monitoring/
│
├── README.md
│
├── Core/
│   ├── Inc/
│   └── Src/
│
├── Drivers/
│
├── STM32_Project/
│
├── Images/
│   ├── hardware.jpg
│   ├── lcd_output.jpg
│   └── uart_output.jpg
│
└── Circuit/
    └── circuit_diagram.png
```

If you are uploading the complete **STM32CubeIDE project**, keep the generated project folders instead of manually copying only `main.c`.

---

# 🧪 Testing and Output

### LCD

The LCD displays:

```text
V:xx.xV I:x.xA
P:xx.xx Watt
```

### UART

At 115200 baud:

```text
LOG: V=5.00, I=0.50, P=2.50
```

### Alert

The alert LED turns ON when the configured voltage or current threshold is exceeded.

---

# 🎯 Applications

The concepts demonstrated by this project can be applied to:

* Energy monitoring systems
* Electrical load monitoring
* Embedded data acquisition
* Equipment monitoring
* Sensor-based measurement systems
* Industrial monitoring prototypes
* IoT energy-monitoring systems

The current project is a **prototype/educational implementation** and would require appropriate isolation, protection, calibration, and certified measurement circuitry before being connected to hazardous mains voltages.

---

# 🚀 Future Improvements

Possible improvements include:

* More accurate sensor calibration
* RMS voltage and current calculation
* Real power and energy measurement
* Power factor calculation
* Multi-channel monitoring
* Graphical display
* SD-card data logging
* Wi-Fi/Bluetooth connectivity
* Cloud-based monitoring
* Fault/event logging
* Improved electrical isolation and protection
* Three-phase monitoring

---

# 🧠 Key Learnings

This project provided practical experience with:

* STM32F446RE microcontroller
* Embedded C programming
* ADC configuration
* DMA-based data acquisition
* UART communication
* GPIO programming
* LCD interfacing
* Sensor interfacing
* Interrupt/callback concepts
* Real-time data processing
* Sensor calibration
* Hardware debugging
* STM32 HAL and CubeIDE

---

