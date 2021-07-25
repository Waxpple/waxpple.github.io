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

# Build a program on FPGAs

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
        m = Module()

        clk_freq = platform.default_clk_frequency

        timer = Signal(int(clk_freq // 2), reset=int(clk_freq // 2) - 1)
        led = platform.request("led").o

        with m.If(timer == 0):
            m.d.sync += timer.eq(timer.reset)
            m.d.sync += led.eq(~led)
        with m.Else():
            m.d.sync += timer.eq(timer -1)
        
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
            Attrs(GLOBAL=True, IO_STANDARD = "LVCMOS33")),
        Resource("led", 0, Pins("B11",dir="o"),
            Attrs(GLOBAL=True, IO_STANDARD = "LVCMOS33")),
        Resource("rst", 0, Pins("L14",dir="i"),
            Attrs(GLOBAL=True, IO_STANDARD = "LVCMOS33")),
    ]
    connectors = []
    def toolchain_program(self, products, name):  
        iceprog = os.environ.get("ICEPROG", "iceprog.exe")
        with products.extract("{}.bin".format(name)) as bitstream_filename:
            subprocess.check_call([iceprog, bitstream_filename])
if __name__ == "__main__":
    Board().build(Blinker(), do_program= False)
```