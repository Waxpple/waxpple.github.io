---
title: "[Day07] Synthesis"
date: 2021-07-18T11:20:21+08:00
draft: true
---
# Supported devices
See the [vendor directory] (https://github.com/m-labs/nmigen/tree/master/nmigen/vendor) for supported devices and toolchain details.

Devices supported as of 18 JUL 2021:

|Device | Platform | Toolchain required|
|-------|----------|-------------------|
|Lattice iCE40||Yosys+nextpnr, LSE-iCECube2, Synplify-iCECube2|
|Lattice MachXO2||Diamond|
|Lattice ECP5||Yosys+nextpnr, Diamond|
|Xilinx Spartan 3A|Xilinx | ISE|
|Xilinx Spartan 6|Xilinx | ISE|
|Xilinx 7-series (Arty, Spartan, Kintex, Virtex)|Xilinx|Vivado|
|Xilinx UltraScale|XilinxUltraScalePlatform|Vivado|
|Intel|IntelPlatform|Quartus|
# Defining your board
Many boards are defined for you at [nmigen_boards](https://github.com/m-labs/nmigen-boards/tree/master/nmigen_boards).

You can copy one from there and modify it to suit you needs, or create a new class subclassed from one of the above supported device platform classes.
For example, I am using [Xilinx-KC705](https://github.com/m-labs/nmigen-boards/blob/master/nmigen_boards/kc705.py) for my development.
## Class properties
- `device`: a string. See the base platform class for which one to choose. This affects options passed to the toolchain so that it compiles for the correct chip.
- `package`: a string. See the base platform class for which one to choose. This affects options passed to the toolchain so that it compiles for the correct package of the chip.
- `resources`: a list of `Resource`. This names the pins you want to use, and configuration options for each such pin.
- `default_clk`: the name of the resource that is the clock for the default clock domain.