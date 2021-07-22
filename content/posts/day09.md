---
title: "[Day10] Formal verification on adder"
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
```bash
sby -f adder.sby
\\output
SBY 16:53:21 [adder_cover] Removing directory 'adder_cover'.                                                                                                                                                         SBY 16:53:21 [adder_cover] Copy 'toplevel.il' to 'adder_cover/src/toplevel.il'.                                                                                                                                      SBY 16:53:21 [adder_cover] engine_0: smtbmc boolector                                                                                                                                                                SBY 16:53:21 [adder_cover] base: starting process "cd adder_cover/src; yosys -ql ../model/design.log ../model/design.ys"                                                                                             SBY 16:53:21 [adder_cover] base: finished (returncode=0)                                                                                                                                                             SBY 16:53:21 [adder_cover] smt2: starting process "cd adder_cover/model; yosys -ql design_smt2.log design_smt2.ys"                                                                                                   SBY 16:53:21 [adder_cover] smt2: finished (returncode=0)                                                                                                                                                             SBY 16:53:21 [adder_cover] engine_0: starting process "cd adder_cover; yosys-smtbmc -s boolector --presat --unroll -c --noprogress -t 2  --append 0 --dump-vcd engine_0/trace%.vcd --dump-vlogtb engine_0/trace%_tb.v --dump-smtc engine_0/trace%.smtc model/design_smt2.smt2"                                                                                                                                                            SBY 16:53:21 [adder_cover] engine_0: ##   0:00:00  Solver: boolector                                                                                                                                                 SBY 16:53:21 [adder_cover] engine_0: ##   0:00:00  Status: passed                                                                                                                                                    SBY 16:53:21 [adder_cover] engine_0: finished (returncode=0)                                                                                                                                                         SBY 16:53:21 [adder_cover] engine_0: Status returned by engine: pass                                                                                                                                                 SBY 16:53:21 [adder_cover] summary: Elapsed clock time [H:MM:SS (secs)]: 0:00:00 (0)                                                                                                                                 SBY 16:53:21 [adder_cover] summary: Elapsed process time [H:MM:SS (secs)]: 0:00:00 (0)                                                                                                                               SBY 16:53:21 [adder_cover] summary: engine_0 (smtbmc boolector) returned pass                                                                                                                                        SBY 16:53:21 [adder_cover] DONE (PASS, rc=0)                                                                                                                                                                         SBY 16:53:21 [adder_bmc] Removing directory 'adder_bmc'.                                                                                                                                                             SBY 16:53:21 [adder_bmc] Copy 'toplevel.il' to 'adder_bmc/src/toplevel.il'.                                                                                                                                          SBY 16:53:21 [adder_bmc] engine_0: smtbmc boolector                                                                                                                                                                  SBY 16:53:21 [adder_bmc] base: starting process "cd adder_bmc/src; yosys -ql ../model/design.log ../model/design.ys"                                                                                                 SBY 16:53:21 [adder_bmc] base: finished (returncode=0)                                                                                                                                                               SBY 16:53:21 [adder_bmc] smt2: starting process "cd adder_bmc/model; yosys -ql design_smt2.log design_smt2.ys"                                                                                                       SBY 16:53:21 [adder_bmc] smt2: finished (returncode=0)                                                                                                                                                               SBY 16:53:21 [adder_bmc] engine_0: starting process "cd adder_bmc; yosys-smtbmc -s boolector --presat --unroll --noprogress -t 2  --append 0 --dump-vcd engine_0/trace.vcd --dump-vlogtb engine_0/trace_tb.v --dump-smtc engine_0/trace.smtc model/design_smt2.smt2"                                                                                                                                                                      SBY 16:53:21 [adder_bmc] engine_0: ##   0:00:00  Solver: boolector                                                                                                                                                   SBY 16:53:21 [adder_bmc] engine_0: ##   0:00:00  Checking assumptions in step 0..                                                                                                                                    SBY 16:53:21 [adder_bmc] engine_0: ##   0:00:00  Checking assertions in step 0..                                                                                                                                     SBY 16:53:21 [adder_bmc] engine_0: ##   0:00:00  BMC failed!                                                                                                                                                         SBY 16:53:21 [adder_bmc] engine_0: ##   0:00:00  Assert failed in top: adder.py:27                                                                                                                                   SBY 16:53:21 [adder_bmc] engine_0: ##   0:00:00  Writing trace to VCD file: engine_0/trace.vcd                                                                                                                       SBY 16:53:21 [adder_bmc] engine_0: ##   0:00:00  Writing trace to Verilog testbench: engine_0/trace_tb.v                                                                                                             SBY 16:53:21 [adder_bmc] engine_0: ##   0:00:00  Writing trace to constraints file: engine_0/trace.smtc                                                                                                              SBY 16:53:21 [adder_bmc] engine_0: ##   0:00:00  Status: failed                                                                                                                                                      SBY 16:53:21 [adder_bmc] engine_0: finished (returncode=1)                                                                                                                                                           SBY 16:53:21 [adder_bmc] engine_0: Status returned by engine: FAIL                                                                                                                                                   SBY 16:53:21 [adder_bmc] summary: Elapsed clock time [H:MM:SS (secs)]: 0:00:00 (0)                                                                                                                                   SBY 16:53:21 [adder_bmc] summary: Elapsed process time [H:MM:SS (secs)]: 0:00:00 (0)                                                                                                                                 SBY 16:53:21 [adder_bmc] summary: engine_0 (smtbmc boolector) returned FAIL                                                                                                                                          SBY 16:53:21 [adder_bmc] summary: counterexample trace: adder_bmc/engine_0/trace.vcd                                                                                                                                 SBY 16:53:21 [adder_bmc] DONE (FAIL, rc=2)                                                                                                                                                                           SBY 16:53:21 One or more tasks produced a non-zero return code.                
```
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
And now the verification result will be:
```bash
SBY 17:00:34 [adder_cover] Removing directory 'adder_cover'.                                                                                                                                                         SBY 17:00:34 [adder_cover] Copy 'toplevel.il' to 'adder_cover/src/toplevel.il'.                                                                                                                                      SBY 17:00:34 [adder_cover] engine_0: smtbmc boolector                                                                                                                                                                SBY 17:00:34 [adder_cover] base: starting process "cd adder_cover/src; yosys -ql ../model/design.log ../model/design.ys"                                                                                             SBY 17:00:34 [adder_cover] base: finished (returncode=0)                                                                                                                                                             SBY 17:00:34 [adder_cover] smt2: starting process "cd adder_cover/model; yosys -ql design_smt2.log design_smt2.ys"                                                                                                   SBY 17:00:34 [adder_cover] smt2: finished (returncode=0)                                                                                                                                                             SBY 17:00:34 [adder_cover] engine_0: starting process "cd adder_cover; yosys-smtbmc -s boolector --presat --unroll -c --noprogress -t 2  --append 0 --dump-vcd engine_0/trace%.vcd --dump-vlogtb engine_0/trace%_tb.v --dump-smtc engine_0/trace%.smtc model/design_smt2.smt2"                                                                                                                                                            SBY 17:00:34 [adder_cover] engine_0: ##   0:00:00  Solver: boolector                                                                                                                                                 SBY 17:00:34 [adder_cover] engine_0: ##   0:00:00  Status: passed                                                                                                                                                    SBY 17:00:34 [adder_cover] engine_0: finished (returncode=0)                                                                                                                                                         SBY 17:00:34 [adder_cover] engine_0: Status returned by engine: pass                                                                                                                                                 SBY 17:00:34 [adder_cover] summary: Elapsed clock time [H:MM:SS (secs)]: 0:00:00 (0)                                                                                                                                 SBY 17:00:34 [adder_cover] summary: Elapsed process time [H:MM:SS (secs)]: 0:00:00 (0)                                                                                                                               SBY 17:00:34 [adder_cover] summary: engine_0 (smtbmc boolector) returned pass                                                                                                                                        SBY 17:00:34 [adder_cover] DONE (PASS, rc=0)                                                                                                                                                                         SBY 17:00:34 [adder_bmc] Removing directory 'adder_bmc'.                                                                                                                                                             SBY 17:00:34 [adder_bmc] Copy 'toplevel.il' to 'adder_bmc/src/toplevel.il'.                                                                                                                                          SBY 17:00:34 [adder_bmc] engine_0: smtbmc boolector                                                                                                                                                                  SBY 17:00:34 [adder_bmc] base: starting process "cd adder_bmc/src; yosys -ql ../model/design.log ../model/design.ys"                                                                                                 SBY 17:00:34 [adder_bmc] base: finished (returncode=0)                                                                                                                                                               SBY 17:00:34 [adder_bmc] smt2: starting process "cd adder_bmc/model; yosys -ql design_smt2.log design_smt2.ys"                                                                                                       SBY 17:00:34 [adder_bmc] smt2: finished (returncode=0)                                                                                                                                                               SBY 17:00:34 [adder_bmc] engine_0: starting process "cd adder_bmc; yosys-smtbmc -s boolector --presat --unroll --noprogress -t 2  --append 0 --dump-vcd engine_0/trace.vcd --dump-vlogtb engine_0/trace_tb.v --dump-smtc engine_0/trace.smtc model/design_smt2.smt2"                                                                                                                                                                      SBY 17:00:34 [adder_bmc] engine_0: ##   0:00:00  Solver: boolector                                                                                                                                                   SBY 17:00:34 [adder_bmc] engine_0: ##   0:00:00  Checking assumptions in step 0..                                                                                                                                    SBY 17:00:34 [adder_bmc] engine_0: ##   0:00:00  Checking assertions in step 0..                                                                                                                                     SBY 17:00:34 [adder_bmc] engine_0: ##   0:00:00  Checking assumptions in step 1..                                                                                                                                    SBY 17:00:34 [adder_bmc] engine_0: ##   0:00:00  Checking assertions in step 1..                                                                                                                                     SBY 17:00:34 [adder_bmc] engine_0: ##   0:00:00  Status: passed                                                                                                                                                      SBY 17:00:34 [adder_bmc] engine_0: finished (returncode=0)                                                                                                                                                           SBY 17:00:34 [adder_bmc] engine_0: Status returned by engine: pass                                                                                                                                                   SBY 17:00:34 [adder_bmc] summary: Elapsed clock time [H:MM:SS (secs)]: 0:00:00 (0)                                                                                                                                   SBY 17:00:34 [adder_bmc] summary: Elapsed process time [H:MM:SS (secs)]: 0:00:00 (0)                                                                                                                                 SBY 17:00:34 [adder_bmc] summary: engine_0 (smtbmc boolector) returned pass                                                                                                                                          SBY 17:00:34 [adder_bmc] DONE (PASS, rc=0) 
```