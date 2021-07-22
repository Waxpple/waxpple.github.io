---
title: "[Day06] Formal_verification"
date: 2021-07-18T09:50:15+08:00
draft: false
---
# Assert, Assume, and Cover for fun and profit.
```python
from nmigen.asserts import Assert, Assume, Cover
from nmigen.cli import main_parser, main_runner
from somewhere import Adder

if __name__ == "__main__":
    parser = main_parser()
    args = parser.parse_args()

    m = Module()
    m.submodules.adder = adder = Adder()
    m.d.comb += Assert(adder.out == (adder.x + adder.y)[:8])
    with m.If(adder.x == (adder.y << 1)):
        m.d.comb += Cover((adder.out > 0x00) & (adder.out < 0x40))
    main_runner(parser, args, m, ports=[] + adder.ports())
```
# Past, Rose, Fell, Stable
```
from nmigen.asserts import Assert, Assume, Cover
from nmigen.asserts import Past, Rose, Fell, Stable
```
# Not ready yet.
