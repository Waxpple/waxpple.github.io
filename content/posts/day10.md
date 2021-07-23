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
We remove simulator part and add `main_runner` into our `clocky.py`.

```python
from typing import List
from nmigen.asserts import Assert,Cover,Assume,Past
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
    with m.If((clocky.x > 0)):
        m.d.sync += Assert(clocky.x == (Past(clocky.x) + 1)[:7])
    main_runner(parser, args, m, ports=[] + clocky.ports())
```

Generate toplevel iLang file and sby file.

```python
python3 clocky.py generate -t il > toplevel.il
```

```python
[tasks]
cover
bmc

[options]
bmc: mode bmc
cover: mode cover
depth 40
multiclock off

[engines]
smtbmc boolector

[script]
read_ilang toplevel.il
prep -top top

[files]
toplevel.il
```

```python
sby -f clocky.sby
```
- Note: use depth 40 which means to run 40 cycles.
The result will be:
![Imgur](https://i.imgur.com/R7Vu8ZD.png)
It Failed on bmc test! Why? We can trace the bmc waveform.

```
gtkwave.exe clocky_bmc/engine_0/trace.vcd &
```
![Imgur](https://i.imgur.com/4GFYE2o.png)
Yes, in formal verification, it found that `Past(x)` is 0x00 and `x` is 0x4A which violates the rule!

Here we change the condition into:

```python
if __name__ == "__main__":
    parser = main_parser()
    args = parser.parse_args()

    m = Module()
    m.submodules.clocky = clocky = Clocky()
    with m.If((clocky.x > 0) & (Past(clocky.load) == 0)):
        m.d.sync += Assert(clocky.x == (Past(clocky.x) + 1)[:7])
    main_runner(parser, args, m, ports=[] + clocky.ports())
```
Assertion will only applied if `x > 0 & Past(load) == 0`. After changing, it will pass the bmc test.
![Imgur](https://i.imgur.com/EIKthPF.png)

Here is another interesting case. We put a assertion like this:

```python
if __name__ == "__main__":
    parser = main_parser()
    args = parser.parse_args()

    m = Module()
    m.submodules.clocky = clocky = Clocky()
    with m.If((clocky.x > 0) & (Past(clocky.load) == 0)):
        m.d.sync += Assert(clocky.x == (Past(clocky.x) + 1)[:7])
    with m.If(clocky.x == 0):
        m.d.sync += Assert(Past(clocky.x) == 100)
    main_runner(parser, args, m, ports=[] + clocky.ports())
```
This statement is not always true if x start with 0x00 at 0 ns. But who knows? Let's fire formal verification up!

```
python3 clocky.py generate -t il > toplevel.il
sby -f clocky.sby
```

![Imgur](https://i.imgur.com/CMAL7EV.png)
Oh, it discoverd our little secret! Let's take a look into trace waveform.

```
gtkwave.exe clocky_bmc/engine_0/trace.vcd &
```

![Imgur](https://i.imgur.com/U6VfrDO.png)
Wow! It actually discovers a specific case that we just talked about. `x` begins with 0 initial value!

Okie, how about we say only checks this condition under `Past(rst) == 0`?

```python
if __name__ == "__main__":
    parser = main_parser()
    args = parser.parse_args()

    m = Module()
    m.submodules.clocky = clocky = Clocky()

    rst = ResetSignal()
    with m.If((clocky.x > 0) & (Past(clocky.load) == 0)):
        m.d.sync += Assert(clocky.x == (Past(clocky.x) + 1)[:7])
    with m.If((clocky.x == 0) & (Past(rst) == 0) ):
        m.d.sync += Assert(Past(clocky.x) == 100)
    main_runner(parser, args, m, ports=[] + clocky.ports())
```

The answer is ... no!

![Imgur](https://i.imgur.com/U6VfrDO.png)