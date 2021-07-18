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