---
title: "[Day09] Formal verification on adder"
date: 2021-07-22T16:27:58+08:00
draft: false
---

# Perform formal verification
What is a "formal verification" ?

`Formal verification` is a process of checking whether a design satisfies some requirements.

# Example
Here is an adder. It is different from previous one on day 8. In order to make sure the adder in the design satisfies "addition". We use assertion to varify the design.
```python
from typing import List
from nmigen.asserts import Assert
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
    m.d.comb += Assert(adder.out == adder.x + adder.y)
    main_runner(parser, args, m, ports=[] + adder.ports())
    # x = Signal(8)
    # y = Signal(8)
    # m.d.comb += adder.x.eq(x)
    # m.d.comb += adder.y.eq(y)
    # sim = Simulator(m)
    # def process():
    #     yield x.eq(0x00)
    #     yield y.eq(0x00)
    #     yield Delay(1e-6)
    #     yield x.eq(0xFF)
    #     yield y.eq(0xFF)
    #     yield Delay(1e-6)
    #     yield x.eq(0x00)
    #     yield Delay(1e-6)
    # sim.add_process(process)
    # with sim.write_vcd("test.vcd", "test.gtkw", traces=[x, y] + adder.ports()):
    #     sim.run()
    # print("end of code")
```
# Generate iLang file
```
python3 adder.py generate -t il > toplevel.il
```
It will generate `toplevel.il` under working directory path.
```python
attribute \generator "nMigen"
attribute \nmigen.hierarchy "top.adder"
module \adder
  attribute \src "adder.py:12"
  wire width 8 output 0 \out
  attribute \src "adder.py:10"
  wire width 8 input 1 \x
  attribute \src "adder.py:11"
  wire width 8 input 2 \y
  attribute \src "adder.py:17"
  wire width 9 $1
  attribute \src "adder.py:17"
  wire width 9 $2
  attribute \src "adder.py:17"
  cell $add $3
    parameter \A_SIGNED 1'0
    parameter \A_WIDTH 4'1000
    parameter \B_SIGNED 1'0
    parameter \B_WIDTH 4'1000
    parameter \Y_WIDTH 4'1001
    connect \A \x
    connect \B \y
    connect \Y $2
  end
  connect $1 $2
  process $group_0
    assign \out 8'00000000
    assign \out $1 [7:0]
    sync init
  end
end
attribute \generator "nMigen"
attribute \top 1
attribute \nmigen.hierarchy "top"
module \top
  attribute \src "adder.py:10"
  wire width 8 input 0 \x
  attribute \src "adder.py:11"
  wire width 8 input 1 \y
  attribute \src "adder.py:12"
  wire width 8 output 2 \out
  cell \adder \adder
    connect \out \out
    connect \x \x
    connect \y \y
  end
  attribute \src "adder.py:27"
  wire width 1 $assert$en
  attribute \src "adder.py:27"
  wire width 1 $assert$check
  attribute \src "adder.py:27"
  wire width 9 $1
  attribute \src "adder.py:27"
  cell $add $2
    parameter \A_SIGNED 1'0
    parameter \A_WIDTH 4'1000
    parameter \B_SIGNED 1'0
    parameter \B_WIDTH 4'1000
    parameter \Y_WIDTH 4'1001
    connect \A \x
    connect \B \y
    connect \Y $1
  end
  attribute \src "adder.py:27"
  wire width 1 $3
  attribute \src "adder.py:27"
  cell $eq $4
    parameter \A_SIGNED 1'0
    parameter \A_WIDTH 4'1000
    parameter \B_SIGNED 1'0
    parameter \B_WIDTH 4'1001
    parameter \Y_WIDTH 1'1
    connect \A \out
    connect \B $1
    connect \Y $3
  end
  attribute \src "adder.py:27"
  cell $assert $5
    connect \A $assert$check
    connect \EN $assert$en
  end
  process $group_0
    assign $assert$en 1'0
    assign $assert$check 1'0
    assign $assert$check $3
    assign $assert$en 1'1
    sync init
  end
end
```
# A sby file for formal verification
```python
[tasks]
cover
bmc

[options]
bmc: mode bmc
cover: mode cover
depth 2
multiclock off

[engines]
smtbmc boolector

[script]
read_ilang toplevel.il
prep -top top

[files]
toplevel.il
```
We will talk about the settings later. Fire the thing up.
![Imgur](https://i.imgur.com/8AroqBb.png)
As you can see, cover test passed but bmc test failed. We can trace it by firing
```
gtkwave.exe adder_bmc/engine_0/trace.vcd
```
![Imgur](https://i.imgur.com/cLjnGam.png)
But yes, 0xFF + 0x01 == 0x00. What is the deal with it? Well, in assertion, we didn't truncate the signal which makes it 8-bit verse 9-bit signal. Here is how we fix our design.
```python
from typing import List
from nmigen.asserts import Assert
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
    # Truncated
    m.d.comb += Assert(adder.out == (adder.x + adder.y)[:8])
    main_runner(parser, args, m, ports=[] + adder.ports())
```
And now the verification result will be correct:
![Imgur](https://i.imgur.com/bmVfoW0.png)

> Note: If you get error message like "no boolector found", go back to day one and install it properly.
# Cover function
```python
from typing import List
from nmigen.asserts import Assert,Cover
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
    m.d.comb += Assert(adder.out == (adder.x + adder.y)[:8])
    m.d.comb += Cover(adder.out == 0xFF)
    m.d.comb += Cover((adder.out == 0xFE) & (adder.x == 0xFE))
    main_runner(parser, args, m, ports=[] + adder.ports())
```
Recompile it and run formal verification.
![Imgur](https://i.imgur.com/al7GTrt.png)
There is two trace file, let's take a look.
```
gtkwave.exe adder_cover/engine_0/trace0.vcd &
```
![Imgur](https://i.imgur.com/PaoN3hy.png)
This is the case one, `out == 0xFF`
```
gtkwave.exe adder_cover/engine_0/trace1.vcd &
```
![Imgur](https://i.imgur.com/CYb1U3n.png)
This is the case two, `out == 0xFE and x == 0xFE`.
Any cover event is independant to any other cover events.

# Assumuption
Assumption is to assume a signal that is not being a illegal state and the signal will always satisfy specific assumptions.
```python
...
if __name__ == "__main__":
    parser = main_parser()
    args = parser.parse_args()
    m = Module()
    
    m.submodules.adder = adder = Adder()
    m.d.comb += Assert(adder.out == (adder.x + adder.y)[:8])
    m.d.comb += Assume(adder.x == (adder.y <<1))
    m.d.comb += Cover((adder.out > 0x00) & (adder.out < 0x40))
    main_runner(parser, args, m, ports=[] + adder.ports())
```
Here we assume `x is always equal to y << 1`. And find one case for `output ranges between 0x00 and 0x40`.
![Imgur](https://i.imgur.com/mwUwgvn.png)
We can do some interesting stuff like:
```python
if __name__ == "__main__":
    parser = main_parser()
    args = parser.parse_args()
    m = Module()
    
    m.submodules.adder = adder = Adder()
    m.d.comb += Assert(adder.out == (adder.x + adder.y)[:8])
    with m.If(adder.x == (adder.y <<1)):
        m.d.comb += Cover((adder.out > 0x00) & (adder.out < 0x40))
    main_runner(parser, args, m, ports=[] + adder.ports())
```
Which means cover the output range if `x == y<<1`.
![Imgur](https://i.imgur.com/II2DFfW.png)