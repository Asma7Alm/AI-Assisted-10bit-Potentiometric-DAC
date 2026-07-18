# AI-Assisted-10bit-Potentiometric-DAC
AI-assisted study and circuit-level implementation of a 10-bit Potentiometric Digital-to-Analog Converter (DAC) using SKY130 PDK and open-source EDA tools.

## Installation of EDA tools used to implement Potentiometric DAC

## Development Environment
The development environment is configured using GitHub Codespaces to perform AI-assisted circuit design and simulation with open-source EDA tools.

#### AI Prompt Log

| Prompt | Outcome |
|--------|---------|
| Should I use a cloud-based Ubuntu environment or a local setup? | Selected GitHub Codespaces. |

## EDA Toolchain

| Tool | Purpose |
|------|---------|
| GitHub Codespaces | Cloud-based Ubuntu development environment |
| ngspice | SPICE circuit simulation |
| xschem | Schematic capture and netlist generation |
| SKY130 PDK | Process Design Kit for SKY130 technology |
| Gaw (optional) | Waveform visualization |

## AI-Assisted Methodology

Artificial intelligence was used to assist with:

- Development environment selection
- Tool installation guidance
- EDA workflow planning
- Circuit-level design guidance
- SPICE simulation support
- Debugging and troubleshooting
  

# Environment Setup – Xschem Symbol Library Fix

## Problem

TG2.sch opened with missing symbols (ipin.sym, pfet_01v8.sym, nfet_01v8.sym, etc.).

## Cause

The user Xschem configuration (~/.xschem/xschemrc) was missing, so Xschem could not locate the SKY130 symbol libraries.

## Solution

` cp /usr/local/share/xschem_sky130/xschemrc ~/.xschem/xschemrc `

## Result

All schematic symbols loaded successfully.

TG2.sch opened correctly.

<img width="1160" height="737" alt="Screenshot 2026-07-15 213904" src="https://github.com/user-attachments/assets/8dba2a12-6a76-4992-993e-cf1592c4f7b9" />

<img width="876" height="587" alt="image" src="https://github.com/user-attachments/assets/9348e614-61e5-4bb8-8229-be34b743f640" />

## TG2 – Transmission Gate

**Objective**

Study the SKY130 transmission gate used as the switching element in the DAC.

**Observations**
TG2 = Transmission Gate (2-input switch).

A transmission gate is an electronic switch that can pass both logic 0 and logic 1 with very low resistance.

It consists of:

- 1 PMOS
- 1 NMOS
- Both connected in parallel
- Controlled by complementary signals (`din` and `dinb`)

**TG2 Architecture**
Total MOSFETs: 8
Control logic: 4 MOSFETs
Transmission gate: 4 MOSFETs

**Purpose**

The control logic generates complementary control signals (din and dinb).
The transmission gate uses these signals to switch the analog input to the output with low resistance.

PMOS (M5)
Source → vdd
Gate → din
Drain → dinb
Bulk → vdd

Function: M5 works with M1 to form a CMOS inverter that generates the complementary control signal

**CMOS Inverter (M5–M1)**
M5 (PMOS) and M1 (NMOS) form a CMOS inverter.
Both gates are driven by din.
Their drains are connected together to produce dinb.
The inverter generates the complementary control signal required by the transmission gate.

**Control Logic**

The TG2 cell contains two CMOS inverter stages:

- Inverter 1 (M5–M1): Generates the complementary signal dinb from din.
- Inverter 2 (M8–M2): Buffers/distributes the control signal to drive the transmission gate transistors.

**The signal flow is:**
din
   │
   ▼
M5 + M1  →  dinb
   │
   ▼
M8 + M2  →  buffered/complementary control
   │
   ▼
Transmission Gate (M3, M4, M6, M7)

**Operation of TG (M6–M4)**

- When din = 1 and dinb = 0, both M6 (PMOS) and M4 (NMOS) turn ON.
- The transmission gate connects inp1 to vout.
- The analog signal is transferred with low resistance and minimal voltage drop.

**Circuit Structure**
- Total MOSFETs: 8
- Control Logic: M1, M2, M5, M8
Transmission Gates:
- M6 + M4 → connect inp1 to vout
- M7 + M3 → connect inp2 to vout
**Working Principle**
- `din = 1`, `dinb = 0` → M6 and M4 turn ON → inp1 is connected to vout.
- `din = 0`, `dinb = 1` → M6 and M4 turn OFF → inp1 is disconnected from vout.

**Key Learning**

A transmission gate uses both an NMOS and a PMOS so that it can pass both LOW and HIGH logic levels (and analog voltages) with low resistance.

## Debugging Log

### Issue 1: Missing TG2 Symbol in `2bitdac.sch`

**Objective:** Open the 2-bit DAC schematic in Xschem.

**Problem:**  
After opening `2bitdac.sch`, three transmission gate blocks appeared as `--- MISSING SYMBOL ---`, so the complete DAC circuit could not be viewed.

**Root Cause:**  
The schematic referenced an absolute symbol path from the original developer's computer:

```text
/home/harshitha/Desktop/xschem/xschem_library/TG2.sym
```

Since this path does not exist in GitHub Codespaces, Xschem could not locate the symbol.

**AI Prompt (Low Token):**

```text
TG2.sym exists but Xschem shows missing symbol. Find the root cause and provide the shortest fix.
```

**Solution:**

Created a backup:

```bash
cp 2bitdac.sch 2bitdac.sch.bak
```

Replaced the absolute path with the local symbol:

```bash
sed -i 's|/home/harshitha/Desktop/xschem/xschem_library/TG2.sym|TG2.sym|g' 2bitdac.sch
```

Reopened the schematic:

```bash
xschem 2bitdac.sch
```

**Result:**  
All three TG2 symbols loaded successfully, and the complete 2-bit DAC schematic became visible.

| Before | After |
|--------|-------|
|<img width="1191" height="811" alt="Screenshot 2026-07-18 121419" src="https://github.com/user-attachments/assets/75a89a75-29b3-4f8d-9a7c-58f580b5f01b" />|<img width="1197" height="855" alt="image" src="https://github.com/user-attachments/assets/31182642-3e6f-4153-b39d-d0ca92b62358" />|

**Observation:**  
Using local symbol references instead of absolute paths makes the schematic portable across different systems and GitHub Codespaces.


## 2-bit DAC Architecture

<img width="1197" height="855" alt="image" src="https://github.com/user-attachments/assets/ee837572-7ff2-4bff-a015-da22997e011e" />

### Key Points

- A 2-bit DAC produces **4 output voltage levels (2² = 4)**.
- The resistor string generates different voltage taps.
- Three transmission gates (TG2) are used to select one of these voltage levels.
- `d0` controls the first stage of selection.
- `d1` controls the final stage and connects the selected voltage to `out_v`.
- This two-stage selection forms a **hierarchical switch network**, which is also used in higher-bit DACs.

### Switch Connections

| Switch | Control | Output |
|--------|---------|--------|
| switch | `d0` | `x1_out` |
| switch1 | `d0` | `x2_out` |
| switch2 | `d1` | `out_v` |

### AI Prompt

```text
Explain the architecture of the 2-bit potentiometric DAC. Help me trace all transmission gates and explain how d0 and d1 select the final output voltage.
```

### Observation

The resistor string creates multiple voltage levels, while the transmission gate network selects one voltage based on the digital inputs (`d0`, `d1`) and sends it to the output.


## Resistor String Analysis

### Key Points

- The resistor string consists of four resistors connected in series.
- Resistor values are:
  - R4 = 500 Ω
  - R1 = 500 Ω
  - R2 = 500 Ω
  - R3 = 250 Ω
- Total resistance of the string is **1750 Ω**.
- The same current flows through all resistors because they are connected in series.
- Voltage drop across each 500 Ω resistor is approximately **0.943 V**.
- Voltage drop across the 250 Ω resistor is approximately **0.471 V**.

### Voltage at Resistor Taps

| Node | Voltage |
|------|---------|
| VREF (`inp1`) | 3.300 V |
| `x1_inp1` | 2.357 V |
| `x1_inp2` | 1.414 V |
| `switch1_inp1` | 0.471 V |
| GND (`inp2`) | 0 V |

### AI Prompt

```text
Help me analyze the resistor string in the 2-bit potentiometric DAC. Calculate the total resistance, current through the resistor ladder, voltage drop across each resistor, and the voltage at every tap.
```

### Observation

The resistor string generates multiple reference voltage levels. Since all resistors are connected in series, the same current flows through each resistor, producing different voltage taps that are later selected by the transmission gate network.


## SPICE Simulation

<img width="1657" height="890" alt="image" src="https://github.com/user-attachments/assets/3f6df73f-0563-4a1c-9c16-1de032ea5f7f" />

### Key Points

- The 2-bit DAC was verified using an NGSPICE transient simulation.
- Digital inputs `d0` and `d1` were applied as pulse signals.
- The output (`out_v`) changed according to the input combinations.
- Four distinct output voltage levels were observed.
- The simulated output closely matched the resistor-string analysis.

### Simulation Setup

- VDD = 3.3 V
- VREF (High) = 3.3 V
- VREF (Low) = 0.1 V
- Simulation: Transient (`.tran 1n 20u`)

### AI Prompt

```text
Help me understand the NGSPICE simulation of my 2-bit DAC. Explain the input pulse signals, output waveform, and compare the simulated output with the resistor-string analysis.
```

### Conclusion

The SPICE simulation confirms that the transmission gate network correctly selects different reference voltages based on the digital inputs, producing multiple analog output levels.


## Building a 3-bit DAC from a 2-bit DAC

<img width="1622" height="926" alt="image" src="https://github.com/user-attachments/assets/b0ab8cd2-df5f-4144-b3f8-3f3f09e35ccc" />

### Commands

```bash
xschem 3bitdac.sch
cat my_3bitdac.spice
```

### Notes

- The 3-bit DAC is built using **two 2-bit DAC blocks**.
- One **TG2** is added to select the final output.
- A new control signal **d2** is introduced.
- A **250 Ω** resistor connects the two 2-bit DAC resistor ladders.
- Two 2-bit DACs provide **8 voltage levels (2³)**.
- This approach is called **hierarchical design**, where larger circuits are built by reusing smaller verified blocks.

AI Discussion:
`How is a 3-bit DAC built using two 2-bit DACs? Why do VLSI designers prefer hierarchical design instead of creating a completely new circuit?`

### Design Insight

Instead of designing a new circuit from scratch, the designer reused the existing 2-bit DAC and added only the required components. This makes the design modular, easier to verify, and scalable to higher-bit DACs.

### Notes

<img width="1470" height="870" alt="image" src="https://github.com/user-attachments/assets/fb8b7f50-7601-47cc-b4bc-7f95339ca902" />

- The waveform verifies the operation of the 3-bit DAC.
- `d0`, `d1`, and `d2` change at different time periods.
- `d2` performs the final selection between the two 2-bit DAC outputs.
- The output changes according to the digital input combinations.


## 10 bit DAC

<img width="1386" height="830" alt="image" src="https://github.com/user-attachments/assets/254cbbaa-ede3-4a3d-a47f-ed2a85e148af" />

### Hierarchical Design

The 10-bit DAC follows the same hierarchical architecture as the 3-bit DAC, where two lower-bit DACs are combined using one TG2 and a 250 Ω resistor. The same design pattern is repeated until the 10-bit DAC is formed.
