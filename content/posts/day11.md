---
title: "[Day11]New toy iCESugar-Pro!"
date: 2021-07-24T21:12:30+08:00
draft: false
---
# iCESugar-pro
![iCESugar-pro](https://www.muselab-tech.com/content/images/2021/03/iCESugar-pro-1.jpg)
Offical github site: [https://github.com/wuxx/icesugar-pro](https://github.com/wuxx/icesugar-pro)

# Install Project Trellis
The FPGA use EPC5 chips from lattice. We can use Trellis+ yosys+ nextpnr to develope our design.
[https://github.com/SymbiFlow/prjtrellis](https://github.com/SymbiFlow/prjtrellis) 

This section will takes more than an hour to make it all done.

## Install the dependencies for Project Trellis

### Install Boost

```bash
sudo apt-get install libboost-all-dev
sudo apt install build-essential libboost-system-dev libboost-thread-dev libboost-program-options-dev libboost-test-dev
sudo apt install  libboost-filesystem1.71-dev
```
### Install cmake

```bash
sudo apt install cmake
```

### Install gcc-9

```bash
sudo apt install gcc-9
```

### Install Eigen3

```bash
sudo apt-get install libeigen3-dev
```

## Install Project trellis

```bash
 git clone --recursive https://github.com/YosysHQ/prjtrellis
cd prjtrellis/libtrellis
cmake -DCMAKE_INSTALL_PREFIX=/usr .
make
sudo make install
```

## Install nextpnr

```bash
git clone https://github.com/YosysHQ/nextpnr
cd nextpnr
cmake . -DARCH=ecp5 -DTRELLIS_INSTALL_PREFIX=/usr .
make -j$(nproc)
sudo make install
```

This step takes long time to make, especially large ram usage. If failed, you can see [https://github.com/YosysHQ/nextpnr/issues/115](https://github.com/YosysHQ/nextpnr/issues/115) for more detailed build information.

# Build a program for FPGAs

```python
import os
import subprocess
from typing import List
from nmigen import Elaboratable, Module, Signal
from nmigen.build import *
from nmigen.build.run import LocalBuildProducts
from nmigen.cli import main_parser, main_runner
from nmigen.vendor.lattice_ecp5 import *

class Blinker(Elaboratable):
    def __init__(self):
        pass
    def elaborate(self, platform: Platform) -> Module:
        led   = platform.request("led", 0)
        timer = Signal(20)

        m = Module()
        m.d.sync += timer.eq(timer + 1)
        m.d.comb += led.o.eq(timer[-1])
        return m
    
    def ports(self) -> List[Signal]:
        return []

class Board(LatticeECP5Platform):
    device = "LFE5U-25F"
    package = "BG256"
    speed = "6"
    default_clk = "clk1"
    default_rst = "rst"
    resources = [
        Resource("clk1", 0, Pins("P6",dir="i"), Clock(25e6),
            Attrs(IO_TYPE = "LVCMOS33")),
        Resource("rst", 0, Pins("L14",dir="i"),
            Attrs(IO_TYPE = "LVCMOS33")),
        Resource("led", 0, Pins("B11",dir="o"),
            Attrs(IO_TYPE = "LVCMOS33")),
    ]
    connectors = []
    def toolchain_program(self, products, name):  
        iceprog = os.environ.get("ICEPROG", "iceprog")
        with products.extract("{}.bin".format(name)) as bitstream_filename:
            subprocess.check_call([iceprog, bitstream_filename])
if __name__ == "__main__":
    Board().build(Blinker(), do_program= False)
```
- Note: In EPC5, IO_STANDARD is IO_TYPE and GLOBAL is not used.

```
python3 blinky.py
```
# Yosys report
In the build folder under python file directory, here is the tree structure.
![Imgur](https://i.imgur.com/W3ykQRW.gif)
```bash
.
├── build_top.bat
├── build_top.sh
├── top.bit
├── top.config
├── top.debug.v
├── top.il
├── top.json
├── top.lpf
├── top.rpt
├── top.svf
├── top.tim
└── top.ys

0 directories, 12 files
```
In `top.rpt`, you can see the report for FPGA utilization.

```bash
=== top ===

   Number of wires:                 23
   Number of wire bits:             99
   Number of public wires:          23
   Number of public wire bits:      99
   Number of memories:               0
   Number of memory bits:            0
   Number of processes:              0
   Number of cells:                 37
     CCU2C                          10
     LUT4                            1
     SGSR                            1
     TRELLIS_FF                     22
     TRELLIS_IO                      3
```
The number of cells is not nessecarily relevant to the code quality. Maybe you change something that you don't think it will impact any design but it does. There is some randomness in synthesis.

In `top.tim`, you can check timing.

```bash
Info: constraining clock net 'clk1_0__io' to 25.00 MHz

Info: Logic utilisation before packing:
Info:     Total LUT4s:        21/24288     0%
Info:         logic LUTs:      1/24288     0%
Info:         carry LUTs:     20/24288     0%
Info:           RAM LUTs:      0/12144     0%
Info:          RAMW LUTs:      0/ 6072     0%

Info:      Total DFFs:        44/24288     0%

Info: Packing IOs..
Info: rst_0__io feeds TRELLIS_IO pin_rst_0.rst_0_0, removing $nextpnr_ibuf rst_0__io.
Info: pin 'pin_rst_0.rst_0_0' constrained to Bel 'X72/Y32/PIOC'.
Info: led_0__io feeds TRELLIS_IO pin_led_0.led_0_0, removing $nextpnr_obuf led_0__io.
Info: pin 'pin_led_0.led_0_0' constrained to Bel 'X49/Y0/PIOA'.
Info: clk1_0__io feeds TRELLIS_IO pin_clk1_0.clk1_0_0, removing $nextpnr_ibuf clk1_0__io.
Info: pin 'pin_clk1_0.clk1_0_0' constrained to Bel 'X0/Y47/PIOC'.
Info: Packing constants..
Info: Packing carries...
Info: Finding LUTFF pairs...
Info: Packing LUT5-7s...
Info: Finding LUT-LUT pairs...
Info: Packing paired LUTs into a SLICE...
Info: Packing unpaired LUTs into a SLICE...
Info: Packing unpaired FFs into a SLICE...
Info: Generating derived timing constraints...
Info: Promoting globals...
Info:     promoting clock net cd_sync_clk1_0__i to global network
Info: Checksum: 0xd8bc23f8

Info: Annotating ports with timing budgets for target frequency 12.00 MHz
Info: Checksum: 0x2027cd80

Info: Device utilisation:
Info:          TRELLIS_SLICE:    16/12144     0%
Info:             TRELLIS_IO:     3/  197     1%
Info:                   DCCA:     1/   56     1%
Info:                 DP16KD:     0/   56     0%
Info:             MULT18X18D:     0/   28     0%
Info:                 ALU54B:     0/   14     0%
Info:                EHXPLLL:     0/    2     0%
Info:                EXTREFB:     0/    1     0%
Info:                   DCUA:     0/    1     0%
Info:              PCSCLKDIV:     0/    2     0%
Info:                IOLOGIC:     0/  128     0%
Info:               SIOLOGIC:     0/   69     0%
Info:                    GSR:     1/    1   100%
Info:                  JTAGG:     0/    1     0%
Info:                   OSCG:     0/    1     0%
Info:                  SEDGA:     0/    1     0%
Info:                    DTR:     0/    1     0%
Info:                USRMCLK:     0/    1     0%
Info:                CLKDIVF:     0/    4     0%
Info:              ECLKSYNCB:     0/   10     0%
Info:                DLLDELD:     0/    8     0%
Info:                 DDRDLL:     0/    4     0%
Info:                DQSBUFM:     0/    8     0%
Info:        TRELLIS_ECLKBUF:     0/    8     0%
Info:           ECLKBRIDGECS:     0/    2     0%
Info:                   DCSC:     0/    2     0%

Info: Placed 4 cells based on constraints.
Info: Creating initial analytic placement for 4 cells, random placement wirelen = 272.
Info:     at initial placer iter 0, wirelen = 112
Info:     at initial placer iter 1, wirelen = 112
Info:     at initial placer iter 2, wirelen = 112
Info:     at initial placer iter 3, wirelen = 112
Info: Running main analytical placer.
Info:     at iteration #1, type TRELLIS_SLICE: wirelen solved = 112, spread = 114, legal = 119; time = 0.00s
Info: HeAP Placer Time: 0.01s
Info:   of which solving equations: 0.00s
Info:   of which spreading cells: 0.00s
Info:   of which strict legalisation: 0.00s

Info: Running simulated annealing placer for refinement.
Info:   at iteration #1: temp = 0.000000, timing cost = 9, wirelen = 119
Info:   at iteration #3: temp = 0.000000, timing cost = 9, wirelen = 115
Info: SA placement time 0.00s

Info: Max frequency for clock '$glbnet$cd_sync_clk1_0__i': 289.52 MHz (PASS at 25.00 MHz)

Info: Max delay <async>                           -> <async>                          : 4.68 ns
Info: Max delay <async>                           -> posedge $glbnet$cd_sync_clk1_0__i: 8.64 ns
Info: Max delay posedge $glbnet$cd_sync_clk1_0__i -> <async>                          : 6.41 ns

Info: Slack histogram:
Info:  legend: * represents 1 endpoint(s)
Info:          + represents [1,1) endpoint(s)
Info: [ 31364,  33877) |*
Info: [ 33877,  36390) |
Info: [ 36390,  38903) |*********************
Info: [ 38903,  41416) |
Info: [ 41416,  43929) |
Info: [ 43929,  46442) |
Info: [ 46442,  48955) |
Info: [ 48955,  51468) |
Info: [ 51468,  53981) |
Info: [ 53981,  56494) |
Info: [ 56494,  59007) |
Info: [ 59007,  61520) |
Info: [ 61520,  64033) |
Info: [ 64033,  66546) |
Info: [ 66546,  69059) |
Info: [ 69059,  71572) |
Info: [ 71572,  74085) |
Info: [ 74085,  76598) |
Info: [ 76598,  79111) |**
Info: [ 79111,  81624) |*
Info: Checksum: 0x607e8c2f
Info: Routing globals...
Info:     routing clock net $glbnet$cd_sync_clk1_0__i using global 0

Info: Routing..
Info: Setting up routing queue.
Info: Routing 58 arcs.
Info:            |   (re-)routed arcs  |   delta    | remaining|       time spent     |
Info:    IterCnt |  w/ripup   wo/ripup |  w/r  wo/r |      arcs| batch(sec) total(sec)|
Info:         58 |        0         58 |    0    58 |         0|       0.02       0.02|
Info: Routing complete.
Info: Router1 time 0.02s
Info: Checksum: 0xe0791b74

Info: Critical path report for clock '$glbnet$cd_sync_clk1_0__i' (posedge -> posedge):
Info: curr total
Info:  0.5  0.5  Source timer$next_CCU2C_S0_9$CCU2_SLICE.Q0
Info:  0.8  1.3    Net timer[0] budget 18.993000 ns (47,2) -> (47,2)
Info:                Sink timer$next_CCU2C_S0_9$CCU2_SLICE.B0
Info:                Defined in:
Info:                  blinky.py:18
Info:                  /usr/local/bin/../share/yosys/ecp5/arith_map.v:63.22-63.23
Info:  0.4  1.8  Source timer$next_CCU2C_S0_9$CCU2_SLICE.FCO
Info:  0.0  1.8    Net timer$next_CCU2C_S0_4_COUT[1] budget 0.000000 ns (47,2) -> (47,2)
Info:                Sink timer$next_CCU2C_S0_3$CCU2_SLICE.FCI
Info:                Defined in:
Info:                  blinky.py:18
Info:                  /usr/local/bin/../share/yosys/ecp5/arith_map.v:63.22-63.23
Info:  0.1  1.9  Source timer$next_CCU2C_S0_3$CCU2_SLICE.FCO
Info:  0.0  1.9    Net timer$next_CCU2C_S0_4_COUT[3] budget 0.000000 ns (47,2) -> (47,2)
Info:                Sink timer$next_CCU2C_S0_2$CCU2_SLICE.FCI
Info:                Defined in:
Info:                  blinky.py:18
Info:                  /usr/local/bin/../share/yosys/ecp5/arith_map.v:63.22-63.23
Info:  0.1  1.9  Source timer$next_CCU2C_S0_2$CCU2_SLICE.FCO
Info:  0.0  1.9    Net timer$next_CCU2C_S0_4_COUT[5] budget 0.000000 ns (47,2) -> (48,2)
Info:                Sink timer$next_CCU2C_S0_1$CCU2_SLICE.FCI
Info:                Defined in:
Info:                  blinky.py:18
Info:                  /usr/local/bin/../share/yosys/ecp5/arith_map.v:63.22-63.23
Info:  0.1  2.0  Source timer$next_CCU2C_S0_1$CCU2_SLICE.FCO
Info:  0.0  2.0    Net timer$next_CCU2C_S0_4_COUT[7] budget 0.000000 ns (48,2) -> (48,2)
Info:                Sink timer$next_CCU2C_S0$CCU2_SLICE.FCI
Info:                Defined in:
Info:                  blinky.py:18
Info:                  /usr/local/bin/../share/yosys/ecp5/arith_map.v:63.22-63.23
Info:  0.1  2.1  Source timer$next_CCU2C_S0$CCU2_SLICE.FCO
Info:  0.0  2.1    Net timer$next_CCU2C_S0_4_COUT[9] budget 0.000000 ns (48,2) -> (48,2)
Info:                Sink timer$next_CCU2C_S0_8$CCU2_SLICE.FCI
Info:                Defined in:
Info:                  blinky.py:18
Info:                  /usr/local/bin/../share/yosys/ecp5/arith_map.v:63.22-63.23
Info:  0.1  2.2  Source timer$next_CCU2C_S0_8$CCU2_SLICE.FCO
Info:  0.0  2.2    Net timer$next_CCU2C_S0_4_COUT[11] budget 0.000000 ns (48,2) -> (48,2)
Info:                Sink timer$next_CCU2C_S0_7$CCU2_SLICE.FCI
Info:                Defined in:
Info:                  blinky.py:18
Info:                  /usr/local/bin/../share/yosys/ecp5/arith_map.v:63.22-63.23
Info:  0.1  2.2  Source timer$next_CCU2C_S0_7$CCU2_SLICE.FCO
Info:  0.0  2.2    Net timer$next_CCU2C_S0_4_COUT[13] budget 0.000000 ns (48,2) -> (49,2)
Info:                Sink timer$next_CCU2C_S0_6$CCU2_SLICE.FCI
Info:                Defined in:
Info:                  blinky.py:18
Info:                  /usr/local/bin/../share/yosys/ecp5/arith_map.v:63.22-63.23
Info:  0.1  2.3  Source timer$next_CCU2C_S0_6$CCU2_SLICE.FCO
Info:  0.0  2.3    Net timer$next_CCU2C_S0_4_COUT[15] budget 0.000000 ns (49,2) -> (49,2)
Info:                Sink timer$next_CCU2C_S0_5$CCU2_SLICE.FCI
Info:                Defined in:
Info:                  blinky.py:18
Info:                  /usr/local/bin/../share/yosys/ecp5/arith_map.v:63.22-63.23
Info:  0.1  2.4  Source timer$next_CCU2C_S0_5$CCU2_SLICE.FCO
Info:  0.0  2.4    Net timer$next_CCU2C_S0_4_COUT[17] budget 0.000000 ns (49,2) -> (49,2)
Info:                Sink timer$next_CCU2C_S0_4$CCU2_SLICE.FCI
Info:                Defined in:
Info:                  blinky.py:18
Info:                  /usr/local/bin/../share/yosys/ecp5/arith_map.v:63.22-63.23
Info:  0.4  2.8  Source timer$next_CCU2C_S0_4$CCU2_SLICE.F0
Info:  0.1  2.9    Net timer$next[18] budget 19.007999 ns (49,2) -> (49,2)
Info:                Sink timer$next_CCU2C_S0_4$CCU2_SLICE.DI0
Info:                Defined in:
Info:                  blinky.py:15
Info:  0.0  2.9  Setup timer$next_CCU2C_S0_4$CCU2_SLICE.DI0
Info: 2.0 ns logic, 1.0 ns routing

Info: Critical path report for cross-domain path '<async>' -> '<async>':
Info: curr total
Info:  0.0  0.0  Source pin_clk1_0.clk1_0_0.O
Info:  1.7  1.7    Net cd_sync_clk1_0__i budget 41.667000 ns (0,47) -> (3,25)
Info:                Sink $gbuf$cd_sync_clk1_0__i.CLKI
Info:                Defined in:
Info:                  /home/waxapple/.local/lib/python3.8/site-packages/nmigen/build/res.py:137
Info:  0.0  1.7  Source $gbuf$cd_sync_clk1_0__i.CLKO
Info:  0.0  1.7    Net $glbnet$cd_sync_clk1_0__i budget 41.666000 ns (3,25) -> (4,49)
Info:                Sink cd_sync.U$$2.CLK
Info: 0.0 ns logic, 1.7 ns routing

Info: Critical path report for cross-domain path '<async>' -> 'posedge $glbnet$cd_sync_clk1_0__i':
Info: curr total
Info:  0.0  0.0  Source pin_rst_0.rst_0_0.O
Info:  3.1  3.1    Net cd_sync_rst_0__i budget 19.882000 ns (72,32) -> (21,33)
Info:                Sink cd_sync.rst_0__i_LUT4_D_SLICE.D1
Info:                Defined in:
Info:                  /home/waxapple/.local/lib/python3.8/site-packages/nmigen/build/res.py:137
Info:  0.2  3.3  Source cd_sync.rst_0__i_LUT4_D_SLICE.F1
Info:  0.1  3.5    Net cd_sync.U$$0_DI budget 19.882000 ns (21,33) -> (21,33)
Info:                Sink cd_sync.rst_0__i_LUT4_D_SLICE.DI1
Info:                Defined in:
Info:                  /home/waxapple/.local/lib/python3.8/site-packages/nmigen/vendor/lattice_ecp5.py:334
Info:  0.0  3.5  Setup cd_sync.rst_0__i_LUT4_D_SLICE.DI1
Info: 0.2 ns logic, 3.2 ns routing

Info: Critical path report for cross-domain path 'posedge $glbnet$cd_sync_clk1_0__i' -> '<async>':
Info: curr total
Info:  0.5  0.5  Source cd_sync.U$$1_SLICE.Q0
Info:  3.0  3.5    Net cd_sync.gsr1 budget 82.807999 ns (21,33) -> (4,49)
Info:                Sink cd_sync.U$$2.GSR
Info:                Defined in:
Info:                  /home/waxapple/.local/lib/python3.8/site-packages/nmigen/vendor/lattice_ecp5.py:330
Info: 0.5 ns logic, 3.0 ns routing

Info: Max frequency for clock '$glbnet$cd_sync_clk1_0__i': 340.48 MHz (PASS at 25.00 MHz)

Info: Max delay <async>                           -> <async>                          : 1.74 ns
Info: Max delay <async>                           -> posedge $glbnet$cd_sync_clk1_0__i: 3.45 ns
Info: Max delay posedge $glbnet$cd_sync_clk1_0__i -> <async>                          : 3.51 ns

Info: Slack histogram:
Info:  legend: * represents 1 endpoint(s)
Info:          + represents [1,1) endpoint(s)
Info: [ 36545,  38816) |*********************
Info: [ 38816,  41087) |*
Info: [ 41087,  43358) |
Info: [ 43358,  45629) |
Info: [ 45629,  47900) |
Info: [ 47900,  50171) |
Info: [ 50171,  52442) |
Info: [ 52442,  54713) |
Info: [ 54713,  56984) |
Info: [ 56984,  59255) |
Info: [ 59255,  61526) |
Info: [ 61526,  63797) |
Info: [ 63797,  66068) |
Info: [ 66068,  68339) |
Info: [ 68339,  70610) |
Info: [ 70610,  72881) |
Info: [ 72881,  75152) |
Info: [ 75152,  77423) |
Info: [ 77423,  79694) |
Info: [ 79694,  81965) |***

Info: Program finished normally.
```

# Program bitstream file into FPGA

![Imgur](https://i.imgur.com/ExHeJTV.gif)