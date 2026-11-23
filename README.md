# KiCad ESP32 demo project, iteration 2 (KiCad 10)

This repository holds the second, corrected iteration of the ESP32 sensor board from my KiCad 9 video lectures, updated and saved in **KiCad 10**. If you want to study the design, modify it or have it manufactured, start here. One known issue remains: automatic reset and download mode do not work, so you upload firmware with the Boot and Reset buttons or through the ESP32-C3's built-in USB. See [Known issue](#known-issue-automatic-reset-and-download-mode-do-not-work).

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
| **[KiCad9-ESP32-demo-project-v1](https://github.com/futureshocked/KiCad9-ESP32-demo-project-v1)** | Iteration 1: the original KiCad 9 files with known errors, and a focused repair of U2, U4 and the microphone output | Follow the original lectures and the repair video, and see how the project developed |
| **KiCad9-ESP32-demo-project-v2-KiCad10** (this one) | Iteration 2, corrected and updated in KiCad 10 | Study, modify or manufacture the board |

I have kept both so that the history of the project stays intact, while anyone who wants to experiment with the design, or have it made, has a clean and correct version to start from.

## Status of the issues found in iteration 1

| Issue | Status in iteration 2 |
|---|---|
| 1. PCB out of sync with the schematic (U2, U4, U5, C15) | **Fixed** |
| 2. Microphone output on an ADC2 pin | **Fixed** |
| 3. Automatic reset and download mode (auto-program circuit) | **Partly fixed, still not working.** Q3 is corrected, but DTR is still connected to an input pin. See [Known issue](#known-issue-automatic-reset-and-download-mode-do-not-work). |

1. **The PCB was out of sync with the schematic.** In iteration 1, nine pads on U4 (CP2102N), U5 (USBLC6), U2 (BME280) and C15 were on +3.3V instead of GND, USB D- or VBUS sense. In this version the PCB was updated from the schematic, the stray +3.3V tracks were removed, and those pads were rerouted to their correct nets.
2. **The microphone output was on GPIO5, an ADC2 pin.** Espressif's ESP32-C3 errata recommends against using ADC2. MIC_OUT is now on GPIO0 (ADC1), and CS_SD moved to GPIO5.
3. **Q3 in the auto-program circuit was wired backwards.** Its collector now connects to BOOT and its emitter to DTR, matching Q2 and Espressif's reference circuit. This correction is necessary but not sufficient, because of the issue described next.

The iteration 1 README describes each problem in detail.

## Known issue: automatic reset and download mode do not work

The auto-program circuit (Q2, Q3, R33, R34) is meant to let the CP2102N's DTR and RTS signals reset the ESP32-C3 and put it into download mode, so that you can upload firmware without pressing any buttons. It needs both DTR and RTS to be outputs from the CP2102N.

The CP2102N in this design is the 20-pin QFN package, which has no DTR pin. The net named DTR is connected to U4 pin 15, which is CTS, an input (see table 5.3 of the CP2102N datasheet in `Datasheets/`). Nothing drives the DTR net, so neither automatic reset nor automatic download mode can be relied on. The same mistake is in iteration 1. DRC cannot detect it, because the schematic and PCB agree with each other.

I have not yet decided whether to change the circuit. Until then, use one of these two methods to upload firmware:

1. **Through the USB-C connector and the CP2102N, with the buttons.** Hold **Boot** (S1), press and release **Reset** (S2), then release **Boot**. The ESP32-C3 starts in download mode and you can upload. Press **Reset** afterwards to run the new firmware. Serial monitoring through the CP2102N works normally.
2. **Through the ESP32-C3's built-in USB Serial/JTAG controller.** The ESP32-C3 has its own USB interface on GPIO18 (D-) and GPIO19 (D+). On this board these pins are brought out to header **J5**: pin 3 is D-, pin 4 is D+ and pin 5 is GND. Wire a USB cable's D-, D+ and GND to those pins and power the board from the USB-C connector or the battery. **Do not connect the cable's 5 V wire to J5**: J5 pin 1 is the 3.3 V rail. Through this interface, esptool and the Arduino and ESP-IDF tools can reset the chip and enter download mode without the buttons, and you get a serial console and JTAG debugging as well. For this to keep working, your firmware must not use GPIO18 and GPIO19 as ordinary I/O.

## Other changes from iteration 1

1. A PTC fuse (F2) on the +5 V output of the charger.
2. A trimmer (R17) to adjust the microphone gain, with new values for R30 (1 kΩ), R31 (22 kΩ) and C19 (10 µF).
3. The light sensor was rearranged so that its output rises with light: Q1's collector goes to +3.3V and R23 pulls the output down to GND.
4. Three extra capacitors: C23 (100 nF decoupling) and C24 and C25 (10 nF filtering on the light sensor signal).
5. The Boot and Reset buttons (S1, S2) were rotated 180°, and the right edge of the board moved about 0.8 mm.

## Design check status

A DRC with **Test for parity between PCB and schematic** enabled reports no errors and no unconnected pads. DRC only compares the PCB with the schematic, so it cannot detect design errors like the DTR problem above. A few warnings remain. None of them affect how the circuit works:

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
