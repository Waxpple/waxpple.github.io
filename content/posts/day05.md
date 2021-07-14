---
title: "[Day05] Simulations"
date: 2021-07-14T15:28:58+08:00
draft: false
---

# Simulating
The best way to simulate a module is through nMigen's `Simulator`.
## Define you ports
Define a `ports` function in your module which returns an array of your module's ports:
```
class YourModule(Elaboratable):
    ...
    def ports(self):
        return [self.youmodule.p1, self.yourmodule.p2, ...]
```
## Create a top-level module
Create a top-level module for your simulation:
```
from nmigen import *
from nmigen.back.pysim import Simulator, Delay, Settle
from somewhere import YourModule

if __name__ == "__main__":
    m = Module()
    m.submodules.yourmodule = yourmodule = YourModule()

    sim = Simulator(m)

    def process():
        # To be defined
    sim.add_process(process) # or sim.add_sync_process(process), see below
    with sim.write_vcd("test.vcd", "test.gtkw", traces=yourmodule.ports()):
        sim.run()
```
> There is currently a bug in nMigen where inputs to your module are not output to the trace file. To get around this, for each such input, place this in your `main` before the `Simulator` construction:
>```
>input1 = Signal()
>m.d.comb += yourmodule.input1.eq(input1)
>...
>sim = Simulator(m)
>```
>Inside your `process`, refer to this input as `input1`, not `yourmodule.input1`. This will force nMigen to include `input1` in the trace file.
