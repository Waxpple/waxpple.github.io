---
title: "[Day13] Writing a simple state machine"
date: 2021-07-30T22:27:14+08:00
draft: false
---

# What is missing?

Actually, when I am writing this article, there is not much tutorial on nmigen. It is not complete in [nmigen tutorial](https://nmigen.info/nmigen/latest/tutorial.html). In this tutorial, I will try my best to elaborate a FSM circuit.

# Finite state machine

Let's say... I want a circuit which samples data every 128+1 clock cycles. Then I need a initial state to reset every signal and one state to add a counter.
`cnt` is a 8-bit counter and `data_in` is a input I/O port and `data_out` is a output I/O port.
If `cnt[8] == 1` which means counting from 0 to 128 and sampling `data_in`.

```python
from nmigen import *
from math import ceil, log2
from nmigen_soc.memory import *
from nmigen_soc.wishbone import *
from typing import List
from nmigen.back.pysim import *

class Simple_FSM(Elaboratable):
    def __init__ (self):
        self.cnt = Signal(8)
        self.data_in = Signal(8)
        self.data_out = Signal(8)

    def elaborate( self, platform):
        m = Module()
        with m.FSM(reset="START"):
            with m.State("START"):
                m.d.sync += self.cnt.eq(0)
                m.d.sync += self.data_out.eq(0)
                m.next = "COUNT"
            with m.State("COUNT"):
                m.d.sync += self.cnt.eq(self.cnt + 1)
                with m.If(self.cnt[-1] == 1):
                    m.d.sync += self.data_out.eq(self.data_in)
                    m.next = "START"
                with m.Else():
                    m.next = "COUNT"
                
                
        return m
if __name__ == "__main__":
    dut = Simple_FSM()
    sim = Simulator(dut)
    def proc():
        for i in range( 200 ):
            yield Tick()
            yield dut.data_in.eq(i%128)
            yield Settle()
            
    sim.add_clock( 1e-6 )
    sim.add_sync_process(proc)

    with sim.write_vcd("test.vcd", "test.gtkw"):
        sim.run()
```

```
python3 Simple_FSM.py
gtkwave.exe test.vcd
```

![Imgur](https://i.imgur.com/eoqr4Yi.png)

# Now maybe we can dive into SPI communication using FSM to establish the protocal.

# COSCUP 

https://github.com/chipsalliance/verible
https://github.com/idea-fasoc/OpenFASOC
https://efabless.com/design_catalog/asic_platform/116
