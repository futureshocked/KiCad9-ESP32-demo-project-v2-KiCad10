# KiCad ESP32 demo project, iteration 2 (KiCad 10)

This repository holds the second, corrected iteration of the ESP32 sensor board from my KiCad 9 video lectures, updated and saved in **KiCad 10**. If you want to study the design, modify it or have it manufactured, start here.

The first iteration, which matches the original lectures and contains the errors described below, is archived in **[KiCad9-ESP32-demo-project-v1](https://github.com/futureshocked/KiCad9-ESP32-demo-project-v1)**.

## About the project

The board is a battery-powered environmental sensor built around the ESP32-C3. It is a four-layer PCB, about 81 × 54 mm, and includes:

1. An ESP32-C3-WROOM-02 module (U3) with Wi-Fi and Bluetooth LE.
2. A USB-C connector with a CP2102N USB-to-UART bridge (U4), USBLC6-2SC6 ESD protection (U5) and automatic boot/reset circuitry (Q2, Q3).
3. An MCP73871 Li-ion charger with power-path management (U1), a PTC fuse on its output (F2) and an LM1117 3.3 V regulator (VR1).
4. A BME280 temperature, humidity and pressure sensor (U2).
5. A TEMT6000 ambient light sensor (Q1).
6. An electret microphone with a MAX4466 amplifier (U7, MK1) and a gain trimmer (R17).
7. A microSD card slot (J3) and a W25Q32 SPI flash chip (U6).
8. A header for an I2C OLED display, status LEDs, and Boot and Reset buttons.

**You need KiCad 10 or later to open these files.** They are saved in the KiCad 10 file format, which KiCad 9 cannot read. The project files are still named `KiCad 9 ESP32 demo project` so that they match the lectures.

## Two repositories, and why

There are two repositories for this project:

| Repository | Contents | Use it to |
|---|---|---|
| **[KiCad9-ESP32-demo-project-v1](https://github.com/futureshocked/KiCad9-ESP32-demo-project-v1)** | Iteration 1, made in KiCad 9, with known errors | Follow the original lectures and see how the project developed |
| **KiCad9-ESP32-demo-project-v2-KiCad10** (this one) | Iteration 2, corrected and updated in KiCad 10 | Study, modify or manufacture the board |

I have kept both so that the history of the project stays intact, while anyone who wants to experiment with the design, or have it made, has a clean and correct version to start from.

## What iteration 2 fixes

Iteration 1 had three problems. All three are fixed here:

1. **The PCB was out of sync with the schematic.** In iteration 1, nine pads on U4 (CP2102N), U5 (USBLC6), U2 (BME280) and C15 were on +3.3V instead of GND, USB D- or VBUS sense. In this version the PCB was updated from the schematic, the stray +3.3V tracks were removed, and those pads were rerouted to their correct nets.
2. **Q3 in the auto-program circuit was wired backwards.** Its collector now connects to BOOT and its emitter to DTR, matching Q2 and Espressif's reference circuit.
3. **The microphone output was on GPIO5, an ADC2 pin.** Espressif's ESP32-C3 errata recommends against using ADC2. MIC_OUT is now on GPIO0 (ADC1), and CS_SD moved to GPIO5.

The iteration 1 README describes each problem in detail.

## Other changes from iteration 1

1. A PTC fuse (F2) on the +5 V output of the charger.
2. A trimmer (R17) to adjust the microphone gain, with new values for R30 (1 kΩ), R31 (22 kΩ) and C19 (10 µF).
3. The light sensor was rearranged so that its output rises with light: Q1's collector goes to +3.3V and R23 pulls the output down to GND.
4. Three extra capacitors: C23 (100 nF decoupling) and C24 and C25 (10 nF filtering on the light sensor signal).
5. The Boot and Reset buttons (S1, S2) were rotated 180°, and the right edge of the board moved about 0.8 mm.

## Design check status

A DRC with **Test for parity between PCB and schematic** enabled reports no errors and no unconnected pads. A few warnings remain. None of them affect how the circuit works:

1. **S1 and S2 parity warnings.** The schematic symbols for the Boot and Reset buttons still name the E-Switch TL3305 footprint, while the PCB uses the TE USLPT2819. The two parts number their pins differently, so DRC reports four pad net conflicts. According to the USLPT2819 datasheet, pins 1 and 2 are connected internally, as are pins 3 and 4, and the switch closes between the two pairs. On the PCB, pins 1 and 2 go to GND and pins 3 and 4 go to BOOT or EN, which is correct for the switch that is actually fitted.
2. **USB D- track width.** D- is routed at 0.254 mm and D+ at 0.2 mm. At USB 2.0 full speed (12 Mbit/s) this makes no practical difference, but the two lines are not a matched differential pair.
3. **Housekeeping warnings.** These are library footprint differences, unmirrored text on the back layer, three logo footprints without schematic symbols, two empty description fields, and an "exclude from BOM" setting on J3 that differs between schematic and PCB.

## Files

1. `KiCad 9 ESP32 demo project.kicad_pro`, `.kicad_sch` and `.kicad_pcb`: the KiCad project, with the sub-sheets `esp32-c3-02.kicad_sch`, `sensors.kicad_sch` and `user_interface.kicad_sch`.
2. `Libraries/`: project-specific symbols, footprints and 3D models.
3. `Gerbers/`: Gerber and Excellon drill files, generated from this PCB with the board's saved plot settings.
4. `BOM2.ods`: the bill of materials.
5. `Datasheets/`: datasheets for the main components.
6. `3Dmodel.stl`: an STL export of the board's 3D model, made in March 2025, before the final KiCad 10 edits.

## License

Copyright © 2025–2026, Peter Dalmaris. Licensed under the CERN-OHL-S v2.0. See [LICENSE](LICENSE).
