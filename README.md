# AI-Assisted-10bit-Potentiometric-DAC
AI-assisted study and circuit-level implementation of a 10-bit Potentiometric Digital-to-Analog Converter (DAC) using SKY130 PDK and open-source EDA tools.


## Project Workflow

The project was completed following a structured engineering workflow:

1. Study the reference repository and resistor-string DAC architecture.
2. Configure the SKY130 open-source EDA environment.
3. Analyze the TG2 transmission gate cell.
4. Verify the 2-bit DAC through schematic inspection and SPICE simulation.
5. Understand hierarchical scaling from 2-bit to 3-bit and 10-bit DACs.
6. Investigate key DAC performance parameters such as resolution, LSB, monotonicity, settling behaviour, INL, and DNL.
7. Document the complete design and verification process.



## Repository Structure
Prelayout/
│
├── TG2.sch
├── 2bitdac.sch
├── 3bitdac.sch
├── ...
├── 10bitdac.sch
│
├── my_2bitdac.spice
├── my_3bitdac.spice
├── my_10bitdac.spice
│
├── Pictures/
│
└── README.md



## Key Features

- Hierarchical resistor-string DAC architecture
- Modular design from 2-bit to 10-bit
- SKY130 Open PDK implementation
- Xschem schematic design
- NGSpice circuit verification
- AI-assisted engineering workflow
- Open-source EDA toolchain


## Technologies Used

| Tool | Purpose |
|------|---------|
| SKY130 PDK | Semiconductor process library |
| Xschem | Schematic capture |
| NGSpice | Circuit simulation |
| GitHub Codespaces | Cloud development |
| Git | Version control |



## Verification Summary

| Module | Verification Method | Status |
|---------|--------------------|--------|
| 2-bit DAC | NGSpice transient simulation | ✅ Passed |
| 3-bit DAC | Hierarchical schematic verification | ✅ Passed |
| 10-bit DAC | Hierarchical design review | ✅ Completed |
| Monotonic behaviour | Output waveform analysis | ✅ Verified |
| R/2 termination | Step-size analysis | ✅ Verified |


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
   \/
M5 + M1  →  dinb
   │
   \/
M8 + M2  →  buffered/complementary control
   │
   \/
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

### Observation-1

The resistor string creates multiple voltage levels, while the transmission gate network selects one voltage based on the digital inputs (`d0`, `d1`) and sends it to the output.


### Observation-2

The resistor ladder consists of three 500 Ω resistors followed by a 250 Ω termination resistor. Unlike a simple resistor divider, the ladder does not end with another 500 Ω resistor. The lower termination resistor (R/2) is intentionally used to properly terminate the resistor string and maintain the intended voltage distribution across the selectable taps.

During simulation, the lower reference node (`vref5`) was observed to be 0.1 V instead of 0 V:

- VREFH = 3.3 V
- VREFL = 0.1 V

This confirmed that the DAC implementation uses a small lower reference voltage together with an R/2 termination resistor to achieve the desired resistor-string behaviour.


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


## Performance Experiment: 2-bit DAC Output Verification

### Objective

To verify that different digital input combinations select different resistor taps and produce distinct analog output voltage levels.

### Method

The 2-bit DAC was simulated using NGSpice with pulse inputs applied to D0 and D1. The output waveform (x1_out_v) was observed while the digital inputs changed through all four possible combinations.

### Observed Results

| Digital Code | Approximate Output Voltage |
|--------------|---------------------------|
| 00 | 0.10 V |
| 01 | 0.56 V |
| 10 | 1.47 V |
| 11 | 2.39 V |

### Observation

The output voltage changes in discrete steps as the digital input changes. Each binary code selects a different tap of the resistor string through the transmission gate network.

The measured output confirms the correct operation of the resistor-string DAC and demonstrates monotonic behaviour, where increasing digital codes correspond to increasing output voltages.

Although this experiment is based on the 2-bit DAC, the same operating principle extends recursively to the higher-bit hierarchical DAC architecture.


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


## Digital Input Control

### Objective

Understand how binary input signals control the switching network of the resistor-string DAC.

### Circuit Understanding

- The DAC receives a 10-bit digital input (D9–D0).
- Each digital input controls the transmission gate network.
- The transmission gates connect one of the resistor-string voltage taps to the output.
- Different binary input combinations produce different analog output voltages.

### Observation

The digital inputs do not generate analog voltages directly. Instead, they control the transmission gates, which select one of the pre-generated voltage levels from the resistor string. This switching mechanism enables accurate digital-to-analog conversion.

### AI Discussion

Explain how binary digital inputs control the transmission gate network in a hierarchical resistor-string DAC and how this determines the analog output voltage.


## Reference Voltage Behaviour

### Objective

Understand how the reference voltages define the analog output range of the resistor-string DAC.

### Circuit Understanding

- The resistor string is connected between VREFH (3.3 V) and VREFL (0 V).
- The resistor ladder generates intermediate voltage levels between these two references.
- The transmission gate network selects one of these voltage levels based on the digital input code.

### Observation

The DAC output is always limited by the applied reference voltages. Since the resistor ladder is connected between 3.3 V and 0 V, the output voltage can only vary within this range. The transmission gates do not generate voltage; they only select an existing resistor tap.

### AI Discussion

Explain how the reference voltages determine the operating range of a resistor-string DAC and why the transmission gates only select existing voltage levels.


## Reference Voltage Behaviour

### Objective

Understand how the reference voltages define the analog output range of the resistor-string DAC.

### Circuit Understanding

- The resistor string is connected between VREFH (3.3 V) and VREFL (0 V).
- The resistor ladder generates intermediate voltage levels between these two references.
- The transmission gate network selects one of these voltage levels based on the digital input code.

### Observation

The DAC output is always limited by the applied reference voltages. Since the resistor ladder is connected between 3.3 V and 0 V, the output voltage can only vary within this range. The transmission gates do not generate voltage; they only select an existing resistor tap.

### AI Discussion

Explain how the reference voltages determine the operating range of a resistor-string DAC and why the transmission gates only select existing voltage levels.


## Output Voltage Step

### Objective

Understand how changes in the digital input code affect the analog output voltage.

### Circuit Understanding

- Each digital input code selects a unique resistor tap through the transmission gate network.
- Adjacent digital codes select adjacent voltage levels in the resistor string.
- The output voltage therefore changes in discrete steps rather than continuously.

### Observation

The resistor-string DAC generates a staircase-like analog output. Every increment in the digital input code moves the output to the next available voltage level. The size of each step depends on the reference voltage range and the DAC resolution.

### AI Discussion

Explain why a resistor-string DAC produces a staircase output waveform and how adjacent digital input codes correspond to adjacent voltage levels.

## Resolution and LSB

### Objective

Understand the voltage resolution of the 10-bit resistor-string DAC.

### Circuit Understanding

- A 10-bit DAC provides \(2^{10}=1024\) output levels.
- The reference voltage range is 0 V to 3.3 V.
- Therefore, the output voltage changes in discrete steps.

### Calculation

Number of output levels:

2¹⁰ = 1024

Ideal LSB:

LSB = 3.3 V / 1024 ≈ 3.22 mV

### Observation

The ideal output voltage increases by approximately 3.22 mV for every increment of one digital code. A higher-bit DAC provides smaller voltage steps, resulting in finer analog resolution.

### AI Discussion

Explain the relationship between DAC resolution, the number of output levels, and the Least Significant Bit (LSB) in a 10-bit resistor-string DAC.



## Monotonicity

### Objective

Understand why the resistor-string DAC produces a monotonically increasing output.

### Circuit Understanding

- The resistor ladder creates an ordered set of voltage levels.
- The transmission gate network selects one resistor tap for each digital input code.
- As the input code increases, the selected tap moves to the same or a higher voltage level.

### Observation

An ideal resistor-string DAC is inherently monotonic because the output always follows the ordered resistor taps. Increasing the digital input code does not produce a lower output voltage.

### AI Discussion

Explain why resistor-string DACs are considered inherently monotonic and how the resistor ladder ensures a predictable increase in output voltage.



## Settling Behaviour

### Objective

Understand how quickly the DAC output reaches its final value after a change in the digital input.

### Circuit Understanding

- A change in the digital input causes the transmission gate network to select a new resistor tap.
- The output voltage requires a short time to stabilize due to switch resistance and parasitic capacitance.
- This stabilization period is known as the settling time.

### Observation

Settling time determines how quickly the DAC can produce an accurate output after a code transition. Faster transmission gates and lower parasitic effects improve the settling performance.

### AI Discussion

Explain the factors that influence the settling time of a resistor-string DAC and why it is important for high-speed digital-to-analog conversion.


## Integral Non-Linearity (INL) and Differential Non-Linearity (DNL)

### Objective

Understand the non-linearity errors that affect the accuracy of a resistor-string DAC.

### Circuit Understanding

- An ideal resistor-string DAC produces equal voltage steps between adjacent output codes.
- Manufacturing variations in resistor values and switch characteristics introduce output errors.
- DNL measures the deviation of each voltage step from the ideal LSB.
- INL measures the deviation of the overall DAC transfer characteristic from the ideal linear response.

### Observation

In an ideal resistor-string DAC, both INL and DNL are close to zero. Resistor mismatches and non-ideal switching increase these errors, reducing the output accuracy of the DAC.

### AI Discussion

Explain the difference between Integral Non-Linearity (INL) and Differential Non-Linearity (DNL) in a resistor-string DAC and discuss how resistor mismatch affects both parameters.



## AI-Assisted Engineering Workflow

Artificial intelligence was used as an engineering assistant throughout the project to:

- Understand resistor-string DAC architecture
- Analyze hierarchical circuit design
- Interpret SPICE simulation results
- Debug Xschem and NGSpice issues
- Improve project documentation

AI served as a technical assistant. All circuit exploration, verification, implementation, debugging, and repository organization were performed by the author.



## References

1. SKY130 Open PDK Documentation
2. Xschem Documentation
3. NGSpice User Manual
4. Caravel User Project Documentation
5. Reference Repository: vsdip/avsddac_3v3_sky130_v1
