# Mecatron_Probation_ROV_RC
Made by Gregory Hans Nugraha.

# Single Digital Input (SPST Switch or Button)

## What is an SPST Switch? 

A regular mechanical switch that can be operated using a button.  

- **SPST** → Single Pole, Single Throw; can only control the current of one circuit (i.e., either on or off).  
- When pressed/triggered, the metal contacts inside come in contact to complete the circuit, thus “turning it on.”

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


| Iteration | Circuit Image | Components Changed | Observations / Issues |
|-----------|---------------|------------------|----------------------|
| 1         | ![Iteration 1](https://github.com/user-attachments/assets/a4ef3a7a-dd8d-4733-85f6-2da897210a51) | Initial setup: SPST + RC debouncing (R=?, C=?) | Minor bouncing, ADC occasionally reads multiple triggers |
| 2         | ![Iteration 2](https://user-images.githubusercontent.com/.../iteration2.png) | Increased C / Adjusted R | Bouncing reduced, response slightly slower |
| 3         | ![Iteration 3](https://user-images.githubusercontent.com/.../iteration3.png) | Added series resistor / diode | ADC stable, minor overshoot on fast presses |
| 4         | ![Iteration 4](https://user-images.githubusercontent.com/.../iteration4.png) | Final adjustments, optimized RC | ADC fully protected, clean signal |


