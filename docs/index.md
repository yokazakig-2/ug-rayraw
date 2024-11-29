# Introduction

## YAENAMI

YAENAMI is an ASIC developed by the KEK IPNS E-sys; it has sophisticated functionalities to readout SiPMs such as bias adjusting DAC, pre-amplifier, shaper, comparator, and waveform digitizer ADC.
The feature of YAENAMI is that it can digitize a incoming signal waveform with 10-bit steps and 100 MSps in maximum.

## RAYRAW

RAYRAW is a general purpose multi-SiPM readout board with 4 YAENAMIs, which readout 32 (8x4) channels.

### Version 1

This was designed as a test bench to evaluate YAENAMI ASICs.
An FPGA firmware dedicated for a triggered-type DAQ system is implemented, but the board has functions to be operated as a trigger-less data-streaming mode.
A part of the board design is common to that of [CIRASAME](https://openit.kek.jp/project/cirasame/cirasame) and [AMANEQ](https://openit.kek.jp/project/StrHRTDC/StrHRTDC).

## Links

- [Open-It CIRASAME project](https://openit.kek.jp/project/cirasame/cirasame)

## Update history

**2024.12**

* Descriptions for Hardware, synchronization, firmware, software, and practical usage

