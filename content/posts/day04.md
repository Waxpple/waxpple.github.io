---
title: "[Day04] Bit selection"
date: 2021-07-13T09:48:38+08:00
draft: false
---
# Spliting and combining signals
## Slicing signals
We can get the least significant bit by `x[0]` or the most significant bit by `x[15]`.
```
>>> from nmigen import *
>>> x = Signal(16)
>>> x
(sig x)
>>> x.shape()
Shape(width=16, signed=False)
>>> x[15]
(slice (sig x) 15:16)
>>> x[15].shape()
Shape(width=1, signed=False)
```
While `x[7:0]` is the way to extract the eight least significant bits in verilog.But in nMigen we use `x[0:8]` or `x[:8]`. It will be `Signal_name[start_bit:bits]`.
```
>>> x[7:0]
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/waxapple/.local/lib/python3.8/site-packages/nmigen/hdl/ast.py", line 238, in __getitem__
    return Slice(self, start, stop)
  File "/home/waxapple/.local/lib/python3.8/site-packages/nmigen/hdl/ast.py", line 659, in __init__
    raise IndexError("Slice start {} must be less than slice stop {}".format(start, stop))
IndexError: Slice start 7 must be less than slice stop 0
>>> x[0:8]
(slice (sig x) 0:8)
>>> x[0:8].shape()
Shape(width=8, signed=False)
```
Remember that since this is Python, negative indices are offsets from the end, so a way of getting the most significant bit is `x[-1]`.
```
>>> x[-1]
(slice (sig x) 15:16)
```
You can use strides: `x[0:8:2]` which is `signal_name[start_bit:bits:strides]`.
```
>>> x[0:8:2]
(cat (slice (sig x) 0:1) (slice (sig x) 2:3) (slice (sig x) 4:5) (slice (sig x) 6:7))
```
Note that taking bits range selection will always result in unsigned signal. \
You can even assign to a piece of a signal:
```
m.d.comb += x[:8].eq(y)
# same as the x[7:0] = {y[7:0]}; in verilog
```
# Tip: Using a slice when comparing
In a situation like this:
```
a = Signal(unsigned(16))
b = Signal(unsigned(16))
c = Signal(unsigned(16))

m.d.comb += c.eq(a+b)
```
We expect that if `a+b` overflows, `c` will just be the lower 16 bits of the result.
```
a = Signal(unsigned(16))
b = Signal(unsigned(16))
z = Signal()

m.d.comb += z.eq((a+b)== 0)
```
However, z will be a 17-bit signal. So a 16-bit overflow is not a 17-bit overflow, and this will result in comparison failure. Such as `a = 0xffff` and `b = 0x0001`, the addition will be `z = 0x10000`, which obviously not zero as we expected. \
Therefore, be careful to slice the result
```
m.d.comb += z.eq((a+b)[:16] == 0)
```
Alternatively, just use an intermediate signal (not recommanded):
```
tmp = Signal(unsigned(16))

m.d.comb += tmp.eq(a+b)
m.d.comb += z.eq(tmp == 0)
```