---
title: "[Day10] Clock domain simulation!"
date: 2021-07-22T23:17:22+08:00
draft: false
---
# Cycle accurate simulation
In previous session, we use combinational circuit design. Now, it is time for some sequential circuits!

```python
from typing import List
from nmigen.asserts import Assert,Cover,Assume
from nmigen import Elaboratable, Module, Signal, Mux
from nmigen.back.pysim import Simulator, Delay
from nmigen.build import Platform
from nmigen.cli import main_parser, main_runner

class Clocky(Elaboratable):
    def __init__(self):
        self.x = Signal(7)
        self.load = Signal()
        self.value = Signal(7)
    def elaborate(self, platform: Platform) -> Module:
        m = Module()

        with m.If(self.load):
            m.d.sync += self.x.eq(Mux(self.value <= 100, self.value, 100))
        with m.Elif(self.x == 100):
            m.d.sync += self.x.eq(0)
        with m.Else():
            m.d.sync += self.x.eq(self.x + 1)
        return m
    def ports(self) -> List[Signal]:
        return [self.x, self.load, self.value]
if __name__ == "__main__":
    parser = main_parser()
    args = parser.parse_args()

    m = Module()
    m.submodules.clocky = clocky = Clocky()

    #main_runner(parser, args, m, ports=[] + clocky.ports())

    load = Signal()
    value = Signal(7)
    m.d.comb += clocky.load.eq(load)
    m.d.comb += clocky.value.eq(value)

    sim = Simulator(m)
    sim.add_clock(1e-6)

    def process():
        yield
        yield load.eq(1)
        yield value.eq(95)
        yield
        yield load.eq(0)
        yield
        yield
        yield
        yield
        yield
        yield
        yield
        yield
        yield
    sim.add_sync_process(process)
    with sim.write_vcd("test.vcd", "test.gtkw", traces=[] + clocky.ports()):
        sim.run()

```

After simulation, here is the waveform.
![Imgur](https://i.imgur.com/6gHk82P.png)
As you can see, `x` will be loaded when `load` is high and after the pulse, `x` will keep increasing each cycle until it reaches 100. Then `x` will be clear.

You might want to ask, why would `x` is not loaded at rising edge of load? That is becuase `load` is changing not simutanously with clock edge but infinitesimally close after clock rising edge. Which also applys to clock falling edge.

# Formal verification