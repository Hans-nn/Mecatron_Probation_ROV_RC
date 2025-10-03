# Mecatron_Probation_ROV_RC
Made by Gregory Hans Nugraha.

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
