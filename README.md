# Rocket Test Stand Sequence Logic

> **Deterministic state machine for Rocket Test Stand automation. Implemented in Structured Text (PLCnext) with 4-phase safety logic for cryogenic propellants and ignition.**

---

## 📺 HMI System Demo
**Watch the full sequence execution and HMI visualization:**


https://github.com/user-attachments/assets/f2ad2a78-e9fc-4fa5-8fe1-44d97f67150f

---

## 📂 Project Files & Documentation
Access the full project resources below:

* [📄 **Full Project Report**](Report.pdf) - Detailed technical breakdown of logic phases and safety constraints.
* [📊 **Presentation Slides**](docs/Rocket%20Test%20Stand%20Technical%20Presentation.pptx) - Project defense and overview.
* [💻 **PLCnext Source Code**](src/PLCNextEngineer_code) - Complete project directory for Phoenix Contact PLCnext Engineer.

---

## 🚀 Project Overview
**Author:** Sinan ONA  
**Institution:** Muğla Sıtkı Koçman University, Electrical & Electronics Engineering  
**Platform:** PLCnext Engineer  
**Language:** Structured Text (IEC 61131-3)

This project implements a **deterministic state machine** for automating a rocket engine test stand. The logic is designed to safely manage high-pressure gases, cryogenic propellants (Liquid Oxygen), and ignition sequencing.

[cite_start]The system utilizes **Structured Text (ST)** to define "Safe Operating Windows" using precise timing (TON) and comparator logic (GT, LT, GE, LE), ensuring that a single fault condition mechanically prevents the sequence from progressing[cite: 284, 287].

---

## 🛡️ Safety Architecture
[cite_start]The control logic is divided into four distinct phases to prevent catastrophic failure[cite: 285]:

### 🛑 Phase 1: Pre-Launch Safety & Arming
[cite_start]Before the system arms, environmental and internal conditions must meet strict "Safe Operating Windows"[cite: 291]:
* [cite_start]**Wind Speed:** Must be between **25.0 – 30.0** units to prevent gas pooling or structural stress[cite: 297].
* [cite_start]**Tank Pressures:** Fuel and Oxidizer tanks must be pressurized between **400 – 500 PSI** to prevent cavitation or bursting[cite: 300, 301].
* [cite_start]**Cockpit Pressure:** Verified $\le$ **14.7 PSI** (1 atm) to ensure safe venting[cite: 310].
* [cite_start]**Nitrogen Supply:** Must be **> 2000 PSI** to guarantee sufficient purge gas availability[cite: 312].

### 💨 Phase 2: System Purge & Stabilization
* [cite_start]**Nitrogen Purge (7.5s):** Forces inert gas through the lines to displace 100% of atmospheric oxygen and moisture[cite: 319, 321].
* [cite_start]**Mechanical Settle (3.0s):** A "dead time" allowing turbulence and water hammer effects to dissipate before sensor readings resume[cite: 323, 324].

### ❄️ Phase 3: Valve Sequencing & Cryogenic Warmup
* [cite_start]**Thermal Interlock:** Logic monitors manifold temperatures[cite: 335].
* **LOX Chill-down:** Delays fuel injection for **10.0s** after the Oxidizer valve opens. [cite_start]This ensures stainless steel manifolds reach cryogenic temps (**-183°C**), keeping the Oxidizer liquid[cite: 330, 333].

### 🔥 Phase 4: Ignition & Run State
* [cite_start]**Igniter Duty Cycle:** Limits spark energy to exactly **1.0s** to prevent coil overheating[cite: 343, 345].
* **Safety Watchdog:** A hard-coded auto-abort timer. [cite_start]If a flame is not detected by the sensor within **1.0s** of fuel valve opening, the system triggers a SCRAM[cite: 346, 348].

---

## 💻 Logic & Implementation
[cite_start]The system relies on **On-Delay Timers (TON)** for sequencing and **Comparison Operators** for interlocks[cite: 287].

### Key Logic Snippet (Structured Text)
```iecst
// --- Phase 1: Safety Checks ---
// Define Safe Operating Windows
Wind_unsafe := GT(Wind_Speed_Sensor, 30.0);
Wind_safe   := LT(Wind_Speed_Sensor, 25.0);
Wind_OK     := Wind_safe AND NOT Wind_unsafe;

Fuel_Pres_OK := GE(Fuel_Pres_sensor, Fuel_Min_Limit) AND LE(Fuel_Pres_sensor, Fuel_Max_Limit);

// Global Safety Latch
Safety_OK := Fuel_Pres_OK AND Cockpit_Pres_OK AND N2_Supply_OK AND Oxidizer_Pres_OK AND Wind_OK;

// --- Phase 3: Cryogenic Chill-down ---
// Ensure manifold is cold before fuel injection
WarmupVar := Oxidizer_Valve AND NOT Temp_safe;
TON3(IN := WarmupVar, PT := T#10s, Q => Ox_Warmup);

Fuel_Valve := Oxidizer_Valve AND (Temp_safe OR Ox_Warmup) AND NOT Abort_Active;




