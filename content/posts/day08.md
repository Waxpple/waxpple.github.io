---
title: "[Day08] First fight with nMigen on simply an adder!"
date: 2021-07-22T15:04:50+08:00
draft: false
---
# Prerequisites
If you would like to perform simulation with GTKwave, a opensource free waveform viewer.
Download the software [here](https://sourceforge.net/projects/gtkwave/)

and put it under `C:/gtkwave`.
Go to the windows path setting and add it under the `$path` variable.
# The main function
We create a simple 8-bit adder
```python
from typing import List

from nmigen import Elaboratable, Module, Signal
from nmigen.back.pysim import Simulator, Delay
from nmigen.build import Platform
from nmigen.cli import main_parser, main_runner

class Adder(Elaboratable):
    def __init__(self):
        self.x = Signal(8)
        self.y = Signal(8)
        self.out = Signal(8)

    def elaborate(self, paltform: Platform) -> Module:
        m = Module()

        m.d.comb += self.out.eq(self.x + self.y)
        return m 
    def ports(self) -> List[Signal]:
        return [self.x, self.y, self.out]
if __name__ == "__main__":
    parser = main_parser()
    args = parser.parse_args()
    m = Module()
    
    m.submodules.adder = adder = Adder()
    
    # main_runner(parser, args, m, ports=[] + adder.ports())
    x = Signal(8)
    y = Signal(8)
    m.d.comb += adder.x.eq(x)
    m.d.comb += adder.y.eq(y)
    sim = Simulator(m)
    def process():
        yield x.eq(0x00)
        yield y.eq(0x00)
        yield Delay(1e-6)
        yield x.eq(0xFF)
        yield y.eq(0xFF)
        yield Delay(1e-6)
        yield x.eq(0x00)
        yield Delay(1e-6)
    sim.add_process(process)
    with sim.write_vcd("test.vcd", "test.gtkw", traces=[x, y] + adder.ports()):
        sim.run()
    print("end of code")
```
# Run the simulation
```
python3 adder.py
```
Remember to type <strong>python3</strong>!
# Get the simulation waveform from GTKwave
After firing simulator, it will generate a file call `test.vcd`.
```
gtkwave.exe test.vcd &
```
The GTKwave will pop up and click adder on the left. Select the signals and right click > recurse import > append.
![Imgur](https://i.imgur.com/CT8hlQR.png)
