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
This becomes especially insidious when combining unsigned and signed signals:
```
ptr = Signal(unsigned(16))
addr = Signal(unsigned(16))
offset = Signal(signed(5))

m.d.comb += ptr.eq(addr + offset)
```
we expect `ptr` to be a 16-bit value, since that is what we set it to be. However, what happens here?
Suppose `addr` is 0 and `offset` is -1. Will this comparison work? No, sorry my dear. It wouldn't work.
Consider that `adder` range from `0x7FFF to 0xFFFF` and `offset` range from `0x7 to 0xF`, which is max((+32767 ~ -32768) + (+7 ~ -8) )= `0x8006`
```
y = Signal()
m.d.comb += y.eq((addr + offset) == 0xFFFF )
```
So the result of `addr+offset` in this is -1, which 2's complement 18-bit is `0x3FFFF`. If we slice it, it will be `0xFFFF`.
# Concatenating signals
You can create a new signal out of other signals using `Cat`:
```
m.d.comb += x.eq(Cat(a, b, ...))
```
This concatenates the given signals *first element last* This is important that a in the example above ends up as the least significant bits of x. That is, the concatenation of `a` and `b` is not `ab` but `ba`. \
It is now easy to swap the bytes of a 16-bit signal:
```
m.d.sync += x.eq(Cat(x[8:], x[:8]))
```
You can also assign to a `Cat`, so swapping the bytes can be accomplished in this way also:
```
m.d.sync += Cat(x[8:], x[:8]).eq(x)
```
# Replicating signals
You can replicate a signal by concatenating it to itself via `Cat(x,x)`. But you can also replicate the signal via `Repl(x,2)`
`Repl` with `Cat` can be used together to, for example, signed-extend a value:
```
uint16 = Signal(unsigned(16)) 
int32 = Signal(signed(32))

m.d.comb += int32.eq(Cat(uint16, Repl(uint16[15],16)))
```
Of course, the same can be done by simply using the right signal types:
```
uint16 = Signal(unsigned(16)) 
int32 = Signal(signed(32))

m.d.comb += int32.eq(uint16)
```
The generated cide will do the right thing.
# Arrays
You can create an array of signals like this:
```
# All of these create an array of 3 16-bit elements:

# Creates an array from a, b, c:
a = Signal(unsigned(16))
b = Signal(unsigned(16))
c = Signal(unsigned(16))

abc = Array([a, b, c])

# Creates an array of 16-bit signals:
x = Array([Signal(unsigned(16)), Signal(unsigned(16)), Signal(unsigned(16)) ])

# Also creates an array of 16-bit signals, taking 
y = Array([Signal(unsigned(16)) for_ in range(3)])
```