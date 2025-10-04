# Mecatron_Probation_ROV_RC
Made by Gregory Hans Nugraha.

# Goals
Create a schematic diagram for a PCB that will accept digital and analog inputs, and output digital signals (LED).

## Instructions
- All digital inputs are to be pulled to **3.3 V** via **100 kΩ** resistors.  
- The schematic diagram must be **easy to read and understand**. Use of labels is expected.  
- All calculations (if any) must be written clearly on the schematic diagram **and** in a separate document.  
- Diagram must be drawn in **KiCad** and submitted as **PDF (plotted, not printed)**.

## Components
Since the hook-up for any category should be the same, only **one set of each** is required.  
Assume each component is connected via a connector; for each element include both:  
1. Component → connector (male) via wires  
2. Connector → PCB (draw this connection on the diagram)

- **Single Digital Input (SPST Switch or Button):** 2 pins  
- **Dual Digital Inputs (SPDT Switch):** 3 pins  
- **Single Analog Input (Potentiometer):** 3 pins  
- **Single Digital Output (LED):** 2 pins

## Extra Credit
- Digital inputs include **debounce circuitry**.  
- Analog inputs include **filtering circuitry**.  
- All inputs: **maximum voltage clamped to Vcc**.  
- Digital outputs (LED) are **driven by MOSFETs** (powered via 5 V); MOSFET gate threshold = **3.3 V**.  
- Digital mutually-exclusive inputs (SPDT switch) should produce outputs **(Signal, isValid)**.
- Given a 5 V supply, derive **3.3 V** on-board using a buck/linear regulator (**AMS1117-3.3**).  
- This board requires proper decoupling for the AMS1117-3.3 — include suitable input/output capacitors on the schematic per the regulator recommendations.

# Single Digital Input (SPST Switch or Button)

## What is an SPST Switch? 

A regular mechanical switch that can be operated using a button.  

- **SPST** → Single Pole, Single Throw; can only control the current of one circuit (i.e., either on or off).  
- When pressed/triggered, the metal contacts inside come in contact to complete the circuit, thus “turning it on.”
<img width="318" height="286" alt="image" src="https://github.com/user-attachments/assets/bfb261b4-7054-4183-aa6e-06971d3af78e" />

**Source:** (GeeksforGeeks, 2025)

---

## The Problem: Bouncing

When an SPST is triggered via a button, the metal contacts **don't stick immediately** but open and close a couple of times before settling. This phenomenon is called **“contact bounce”** and can cause the **Microcontroller Unit (MCU)** to read multiple false signals.
**Source:** (GeeksforGeeks, 2025)
---

## Solution: Debouncing

There are multiple ways of debouncing. According to (Maxfield, 2020), there are multiple methods to debounce, including the use of logic gates. However, while these solutions might work, the use of NAND gates **complicates the circuit and increases costs**.  

For this project, components that involved SPST and SPDT switches will be debounced using **capacitors** instead.  

Accordingt to (Mackey, 2024), Ideally, an **oscilloscope** is used to calculate the duration of the bounces of a switch. From the measured duration, a **time constant \(T\)** greater than the bounce duration is selected. This value can then be used in:

$$
T = R \cdot C
$$

to calculate the required capacitance, assuming the resistance \(R\) is known. 

However, since the specific switch, and subsequently its specifications are yet to be determined, For subsequent switches that require debouncing, a safe estimate for $(T = 10 \, \text{ms} = 0.01 \, \text{s}\)$ is used, which is greater than the typical bounce duration *(Real Electronics, 2025)*.  

Thus, using $\(T = R \cdot C\), for \(R = 100\,\text{k}\Omega\)$, the value of \(C\) is:

$$
C = \frac{T}{R} = 0.1 \, \mu\text{F}
$$

## Pull-Up Resistor
Pull-up resistors are simply fixed-value resistors connected between the voltage supply (typically +5 V, +3.3 V, or +2.5 V) and the appropriate pin.

This is done to ensure that a wire is pulled to a high logical level in the absence of an input signal.

## Iterations
| Iteration | Circuit Image | Components Changed | Observations / Issues |
|-----------|---------------|------------------|----------------------|
| 1         | ![Iteration 1](https://github.com/user-attachments/assets/a4ef3a7a-dd8d-4733-85f6-2da897210a51) | Initial setup: SPST + RC debouncing (R=100k Ohms, C=100 μF) | Wrong Capacitance Value, used wrong unit for T, grounded the switch instead  |
| 2         | ![Iteration 2](https://github.com/user-attachments/assets/9dad318a-9044-4730-85f2-1df5f5f30fec) | Increased C / Adjusted R | Corrected circuitry and capacitance value |
| 3         | ![Iteration 3](https://github.com/user-attachments/assets/ceaaa4bd-b82f-454f-b5ea-f80f0c548bd8) | 1. Added diode for voltage clamping (Process is elaborated below) 2. Changed labeling for clarity|

# Dual Digital Inputs (SPDT Switch): 3 Pins

## What is an SPDT Switch?  
It is essentially a switch, similar to an SPST. However, instead of having just one throw (output), it has two.  

For this project, the SPDT is assumed to be used more as a **“mode selector”**, acting differently from an SPST switch.  

---

## Choosing the Switch  
In KiCAD 9.0, there are many versions of SPDT switches. The main requirement is that the switch needs to be **physically actuated**.  

- Used: `SW_Push_SPDT`  
- Not used: `SW_Reed_SPDT` (activates near a magnetic field)  

---

## Debouncing  
Since both throws are used, **both throws need to be debounced** using the same method explained in the SPST section.  

Because the pull-up resistor’s value is the same, the capacitor value will also be the same.  

---

## Pull-Up Resistors  
Pull-up resistors are simply fixed-value resistors connected between the voltage supply (typically +5 V, +3.3 V, or +2.5 V) and the appropriate pin.  

This ensures that a wire is pulled to a **high logical level** in the absence of an input signal.  

---

## Digital Mutually Exclusive Inputs (SPDT Switch) → Output (`Signal`, `isValid`)  
- The input is **not valid** when both throws are triggered (i.e., during bouncing).  
- The input is considered **valid if only one of the throws is triggered**.  

A logic gate can be used to determine this. According to *GeeksforGeeks (2025)*, multiple logic gates exist, but the suitable one here is **XOR**.  

The XOR output is high only when **exactly one contact (NO or NC) is closed**, and low when both are active or both are inactive.  

---

## XOR Gate Selection in KiCad  
In KiCad, two nearly identical XOR gates were considered:  

| Feature | 74AUC2G86 | 74LVC2G86 |
|---------|-----------|-----------|
| **Supply voltage (Vcc)** | 0.8–3.6 V | 1.65–5.5 V |
| **Propagation delay (t_PD)** | ~0.7–1.5 ns | ~2–3 ns |
| **Input HIGH voltage (V_IH)** | Lower, optimized for very low Vcc | Slightly higher |
| **Output drive (I_O)** | Low to moderate | Higher than AUC, can drive more fanout |
| **ESD & robustness** | Less tolerant to overvoltage | More tolerant, robust in standard circuits |
| **Cost & availability** | Usually higher | Cheaper, widely available |

---

## Final Consideration  
The choice here is the **74LVC2G86**, due to its availability and robustness.  

Although it is slightly slower (by ~0.5 ns), it is more robust, cheaper, and widely available. Since this circuit is used for an **RC remote**, switching speed is not the main priority, making the small delay negligible.

## Iterations
| Iteration | Circuit Image | Components Changed | Observations / Issues |
|-----------|---------------|------------------|----------------------|
| 1         | ![Iteration 1](https://github.com/user-attachments/assets/ea80ab55-81bb-4ebe-b276-39443841ad70) | Initial setup of the SPDT switch: pull up 100k ohm resistor, 3.3v from buck converter, and 2 NAND gates | Adds an unesecarry layer of complexity and is also more expensive  |
| 2         | ![Iteration 2](https://github.com/user-attachments/assets/d0b93334-8268-495a-9fc5-7e417f4c32c8) | Changed debounce circuitry using RC, same values as SPST | Improved simplicity and changed labeling for clarity |
| 3         | ![Iteration 3](https://github.com/user-attachments/assets/4b190c3b-2768-4b10-86b7-97b0496025b3) | Added XOR gate for isValid logic and | - |
| 4         | ![Iteration 4](https://github.com/user-attachments/assets/55b7cec0-5aff-421c-835e-e608aabb2051) | 1. Added diode for voltage clamping (Process is elaborated below) 2. Changed labeling for clarity| | - |

# Single Analog Input (Potentiometer): 3 Pins  

## What is a Potentiometer?  
In essence, a potentiometer is a variable resistor. When the knob is rotated, the resistance changes, thus the output voltage also changes. For ESP-32, the common value for potentiometers is 10k Ohm. That will be used for this project. 

### Problems  
In reality, the output may jump slightly due to mechanical contacts (the wiper) or electrical noise. This results in signal fluctuations or “jitter,” especially in digital systems reading analog values, such as the ADC in a microcontroller.  

---

## Types of Filtering Circuitry  
There are many types of filtering circuitry:  
- Low-Pass Filter (LPF)  
- High-Pass Filter (HPF)  
- Band-Pass Filter (BPF)  
- Band-Stop Filter (BSF) or Notch Filter  

To properly determine the type of filtering circuitry, we must see the source of the problem we are trying to solve.  

- Signal fluctuations or “jitter” due to mechanical contacts or noise are **high-frequency spikes or rapid changes**.  
- Meanwhile, the potentiometer is supposed to **gradually change voltage** (the intended frequency here is low).  

Thus, the filter needs to filter out high-frequency noise and only let low-frequency signals pass through → **Low-Pass Filter (LPF).**  

Since the frequency of the voltage change comes from human action (turning the knob), as long as the cutoff frequency is above the maximum frequency of human knob turning, it should be fine.  

---

## Low-Pass Filter Cutoff Frequency  

The cutoff frequency of a low-pass filter is given by:  

$$
f_c = \frac{1}{2 \pi R C}
$$  

Where:  
- \( f_c \): Cutoff frequency (Hz)  
- \( R \): Resistance (Ohms, Ω)  
- \( C \): Capacitance (Farads, F)  

From this formula, both the cutoff frequency and the capacitor value must be determined.  

---

## Cutoff Frequency Consideration  
According to *ValdPerformance (2025)*, the maximum frequency from a human-adjusted knob is 20 Hz.  
However, since 20 adjustments per second (20 Hz) is too high for a remote control, **10 Hz will be used** as it is closer to the maximum realistic speed for this intended use.  

---

## Determining Resistance Seen by the Capacitor  
In this case, I searched on Google and found that for ESP32s, the common convention is to use **10 kΩ potentiometers** for ADCs (to be confirmed).  

When calculating the cutoff frequency, the resistance used is the **worst-case resistance / largest R seen by the capacitor**.  
Since the capacitor is connected to the wiper, the resistance “seen” is basically \( R_{12} \parallel R_{23} \).  

Thus, the largest \( R \) would be when the wiper is at the middle:  

$$
R = \frac{R_{\text{pot}}}{2} = \frac{10kΩ}{2} = 5kΩ
$$  

---

## Capacitance Value  
Using the formula above, the capacitance can be determined:  

$$
C = \frac{1}{2 \pi R f_c}
$$

Substituting \( R = 5kΩ \) and \( f_c = 10Hz \):  

$$
C = \frac{1}{2 \pi (5000)(10)} \approx 3.18 \, \mu F
$$

---

## Final Parameters  

| Parameter       | Value     | Notes                                      |
|-----------------|-----------|--------------------------------------------|
| Resistance (R)  | 5 kΩ      | Worst-case wiper position (midpoint)       |
| Cutoff freq (f) | 10 Hz     | Based on expected knob turning speed       |
| Capacitance (C) | 3.18 μF   | Calculated using LPF cutoff frequency      |

## Iterations
| Iteration | Circuit Image | Components Changed | Observations / Issues |
|-----------|---------------|------------------|----------------------|
| 1         | ![Iteration 1](https://github.com/user-attachments/assets/42be2d1c-e50d-482f-84bb-fa20674d96f2) | Initial setup of potentiometer | Forgot to connect pin 3 to GND, no voltage clamp yet |
| 2         | ![Iteration 2](https://github.com/user-attachments/assets/3c9894a6-22f9-43a8-b03b-bcce31f4780e) | Added diode for voltage clamping | - |

# Single Digital Output (LED): 2 Pins  

## Specifications of the resistor
To avoid damaging the LED, a maximum current needs to be set. The convention for maximum current is **0.02 A (20 mA)** (RoboticsBack-End, n.d.).  

LEDs have a **forward voltage (voltage drop)**, which varies depending on color but can generally be approximated as **2 V**.  

The resistor can be calculated as:  

$$
R = \frac{V_{source} - V_{forward}}{I} = \frac{5V - 2V}{0.02A} = 150 \, \Omega
$$  

This is the minimum value; a higher resistance is also acceptable since it will keep the current below 20 mA.  
Because 150 Ω resistors are uncommon, **220 Ω** will be used.  

---

## Digital Outputs (LED) Driven by MOSFETs (via 5V), MOSFET Gate Threshold = 3.3V  

### What is a MOSFET?  

By definition, a MOSFET is a high-power, fast switch (Electronics-Tutorial, n.d.).  

There are two types of MOSFETs:  
- **NMOS** – Activates when V<sub>gate</sub> is above V<sub>threshold</sub>.  
- **PMOS** – Activates when V<sub>gate</sub> is below V<sub>threshold</sub>.  

Since the ESP32 uses 3.3 V logic to activate the gate, an **NMOS** will be used.  

In short, an NMOS is just an MCU-controllable switch. When the GPIO outputs 3.3 V (above threshold), current will flow from **drain to source**.  

---

# NMOS Comparison: 2N7002 vs AO3400  

| Feature                  | 2N7002                            | AO3400                             |
|---------------------------|-----------------------------------|------------------------------------|
| Type                     | N-channel MOSFET                  | N-channel MOSFET                   |
| V<sub>DS</sub> (Drain-Source Voltage) | 60 V max                        | 30 V max                           |
| I<sub>D</sub> (Continuous Drain Current) | ~200 mA (at 25 °C)              | ~5.8 A (at 25 °C)                  |
| R<sub>DS(on)</sub> @ V<sub>GS</sub>=2.5V | ~7.5 Ω                          | ~35 mΩ                             |
| R<sub>DS(on)</sub> @ V<sub>GS</sub>=4.5V | ~2.0 Ω                          | ~22 mΩ                             |
| Gate Threshold V<sub>GS(th)</sub> | 0.8–3.0 V                        | 1.0–3.0 V                          |
| Total Gate Charge (Qg)   | ~2.5 nC                           | ~9–12 nC                           |
| Package                  | SOT-23                            | SOT-23                             |
| Use Case                 | Logic-level signals, light loads  | Power switching, higher current     |
| Cost & availability      | Very cheap, widely available      | Slightly more expensive, widely used |

---

Both of these NMOS devices have a threshold of approximately **1–2.5 V**, making it easy for the 3.3 V logic from the ESP32 to surpass the threshold and activate the gate. However, since the **2N7002** is much cheaper and widely available, it is the preferred choice for this application.  

## Iterations
| Iteration | Circuit Image | Components Changed | Observations / Issues |
|-----------|---------------|------------------|----------------------|
| 1         | ![Iteration 1](https://github.com/user-attachments/assets/68dc26d4-2431-4db7-91cb-d001538a9403) | Initial setup of the LED, 100 ohm resistor (wrong values), without voltage clamp, and no MOSFAT|
| 2         | ![Iteration 2](https://github.com/user-attachments/assets/7ed1bc35-13eb-4cd2-9b70-924e584fd5df) | Added Mosfat, changed resistor values (re-calculated) | There is still potential for floating values, because no pull down resistor, resistor value is still at minimum, possibility of voltage-inrush (need voltage clamp)  |
| 3         | ![Iteration 3](https://github.com/user-attachments/assets/ef186cb6-3199-484f-aa4e-be2780487e47) | Changed resistor value to 220 (standard value) and so that its not at a minimum, added voltage clamp through diode | - |

# 3.3 Volt Buck Converter  

**Source:** [IC Components – AMS1117-3.3 Features, Applications, and Wiring Guide](https://www.ic-components.com/blog/ams1117-3.3-features,applications,and-wiring-guide-for-stable-voltage-regulation.jsp)  

---

## What is a Buck Converter?  
An **AMS1117-3.3** is a voltage regulator. It works by rapidly switching the input DC voltage on and off.  

---

## Main Problems / Issues (Analog Devices, 2002)  
While switching on and off is effective for regulating voltage, it introduces **high-frequency noise**.  

Additionally, sudden load changes (e.g., turning on an LED) require the converter to supply extra current to meet demand. Without proper filtering, this can cause **voltage dips** or **instability**.  

---

## Solution: Capacitor for Wide-Band Filtering  

For the AMS1117-3.3V LDO:  
- Use **0.1 μF** capacitor for high-frequency filtering.  
- Use **10 μF** capacitor for low-frequency filtering.  
- Together, these form a **wide-band filter**.  

Since the capacitors are connected in parallel, the order (left to right) does not matter, as long as there is a **pair of 10 μF and 0.1 μF capacitors** on each side.

## Iterations
| Iteration | Circuit Image | Components Changed | Observations / Issues |
|-----------|---------------|------------------|----------------------|
| 1         | ![Iteration 1](https://github.com/user-attachments/assets/ac92b45d-dd68-4f3b-a961-d818c2171902) | Initial Setup of the AMS1117-3.3V LDO, with wide-band filtering from input and output | -|
| 2         | ![Iteration 2](https://github.com/user-attachments/assets/46fff5e6-7ba9-45fc-9e57-730db8b55ea9) | Changed output 10 μF capacitor to 22  μF (based on datasheet) | -|

# Voltage Clamp

## What is Voltage Clamping?  
According to (Electrical4U, 2021), clamping voltage is the maximum voltage permissible through a surge protector (i.e., a diode in our case). Any voltage above this maximum is limited, thereby protecting connected devices.

---

## Types of Diodes  

### Comparison: Reverse-Biased Zener Diode vs TVS Diode  

| Feature / Aspect                  | Reverse-Biased Zener Diode                        | TVS (Transient Voltage Suppression) Diode         |
|----------------------------------|--------------------------------------------------|--------------------------------------------------|
| **Purpose**                       | Voltage regulation / simple overvoltage protection | Designed specifically for surges and voltage spikes |
| **Connection**                    | Across Vcc and GND (cathode to Vcc, anode to GND) | Across supply rails or input signals             |
| **Normal Operation**              | Does nothing                                     | Does nothing                                     |
| **Response to Voltage Spike**     | Conducts when voltage exceeds Zener rating        | Conducts extremely fast (nanoseconds) to high-voltage spikes |
| **Voltage Selection**             | Zener voltage slightly above normal Vcc           | Clamping voltage chosen to protect sensitive components |
| **Directionality**                | Typically unidirectional                          | Can be unidirectional or bidirectional           |
| **Speed**                         | Slower response than TVS                          | Very fast response (ns range)                    |
| **Use Case Notes**                | Simple overvoltage protection for power rails     | Protects circuits from fast transients and surges |

---

In this case, a **TVS diode** is chosen due to its superior performance against fast spikes caused by switching, ESD, or inductive loads. It is also the **common/conventional solution** when clamping voltages.

---

## TVS Diodes  
According to (Toshiba, n.d.), TVS diodes are commonly connected between the signal line and GND of externally exposed terminals (e.g., USB). This is because harmful **Electrostatic Discharge (ESD)** — the sudden discharge of static electricity — may occur through these points.

---

**Image of circuitry and concept of voltage clamping:**  
<img width="731" height="260" alt="image" src="https://github.com/user-attachments/assets/26bed114-b04d-4731-93d2-1a6131debdb7" />
Source: (Toshiba, n.d.)


Connecting to ESP32-S3
Based on datasheet: (https://www.espressif.com/sites/default/files/documentation/esp32-s3-wroom-1_wroom-1u_datasheet_en.pdf). Summarized by chatGPT.
# Connecting to ESP32

## 1. GPIOs you must avoid (boot strapping pins)
- **GPIO 0, 2, 15** → decide boot mode. Pulling them wrong = ESP32 won’t boot.  
- **GPIO 6–11** → connected to flash, don’t touch.  
- **GPIO 34–39** → input-only (no output).  

## 2. Good choices for general I/O

### Digital Inputs / Buttons (SPST, SPDT)
- Use GPIOs: `16, 17, 18, 19, 21, 22, 23, 25, 26, 27, 32, 33`

### Analog Input (Potentiometer)
- Use ADC-capable pins: `32, 33, 34, 35, 36, 39`  
  *Note: GPIO 34–39 are input-only.*

### Digital Output (LED via MOSFET)
- Use PWM-capable pins: `18, 19, 21, 22, 23, 25, 26, 27, 32, 33`

| Function               | Net Label       | ESP32 Pin | Notes                          |
| ---------------------- | --------------- | --------- | ------------------------------ |
| Single Button (SPST)   | SPST_DIGIN1     | GPIO18    | Pulled up, optional debounce   |
| SPDT Throw A           | SPDT1_DIGIN2    | GPIO13    | Input to XOR                   |
| SPDT Throw B           | SPDT2_DIGIN3    | GPIO17    | Input to XOR                   |
| XOR Output (`isValid`) | IS_Valid_DIGIN4 | GPIO14    | Hardware-computed via XOR gate |
| LED Output             | NMOS_DIGOUT1    | GPIO21    | Drives NMOS gate, ON/OFF only  |
| Potentiometer ADC      | ADC_POT         | GPIO33    | Analog input                   |

From Datasheet
| Pin        | Type  | Domain / Notes                         |
| ---------- | ----- | -------------------------------------- |
| VDD3P3     | Input | Analog power domain                    |
| VDD3P3_RTC | Input | RTC and part of digital power domains  |
| VDD_SPI    | Input | Flash/PSRAM memory (backup power line) |
| VDD3P3_CPU | Input | Digital power domain                   |
| VDDA       | Input | Analog power domain (ADC reference)    |
| GND        | –     | External ground connection             |

Thus, VDD3P3, VDD3P3_CPU, VDDA, VDD3P3_RTC need to be all connected to 3.3V from AMS1117-3.3V LDO and GND. 

** FINAL ESP 32 CONNECTIONS ** 
<img width="540" height="582" alt="image" src="https://github.com/user-attachments/assets/a3bf0885-6c15-448d-9a88-fe63231e3037" />

