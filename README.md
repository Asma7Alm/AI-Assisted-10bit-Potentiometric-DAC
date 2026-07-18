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
| *(Insert screenshot with missing TG2 symbols)* | *(Insert screenshot with corrected schematic)* |

**Observation:**  
Using local symbol references instead of absolute paths makes the schematic portable across different systems and GitHub Codespaces.
