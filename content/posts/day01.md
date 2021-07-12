---
title: "[Day01] Basic nMigen"
date: 2021-07-10T10:50:02+08:00
draft: false
---
# Basic structure of modules
```
from nmigen import *
from nmigen.build import Platform

class ThingBlock(Elaboratable):
    def __init__(self):
        pass
    def elaborate(self, platform: Platform) -> Module:
        m = Module()
        return m
```
# Elaborating a module
```
from nmigen.cli import main

if __name__== "__main__":
    sync = ClockDomain()
    block = ThingBlock()

    m = Module()
    m.domains += sync
    m.submodules += block

    main(m, ports=[sync.clk,sync.rst])
```
- main(module, ports=[<ports>], platform="<platform>") translate the given module into verilog. This is call *elaboration*. All elaborate() medthod will have its platform argument set to the given platform like particular chips or evaluation boards.
```
python3 thing.py generate -t [v|il] > thing.[v|il]
```
> If you encounter any error message, Back to day00 and install the pre-requisties.
- Choose RTLIL if using yosys.
