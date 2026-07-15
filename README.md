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

##Problem

TG2.sch opened with missing symbols (ipin.sym, pfet_01v8.sym, nfet_01v8.sym, etc.).

##Cause

The user Xschem configuration (~/.xschem/xschemrc) was missing, so Xschem could not locate the SKY130 symbol libraries.

##Solution

` cp /usr/local/share/xschem_sky130/xschemrc ~/.xschem/xschemrc `

## Result

All schematic symbols loaded successfully.
TG2.sch opened correctly.

<img width="1160" height="737" alt="Screenshot 2026-07-15 213904" src="https://github.com/user-attachments/assets/8dba2a12-6a76-4992-993e-cf1592c4f7b9" />

<img width="876" height="587" alt="image" src="https://github.com/user-attachments/assets/9348e614-61e5-4bb8-8229-be34b743f640" />
