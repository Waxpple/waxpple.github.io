---
title: "[Day04] Bit selection"
date: 2021-07-13T09:48:38+08:00
draft: false
---
# Spliting and combining signals
## Slicing signals
We can get the least significant bit by `x[0]` or the most significant bit by `x[15]`.
```python
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
```python
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
```python
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
```python
m.d.comb += x[:8].eq(y)
# same as the x[7:0] = {y[7:0]}; in verilog
```
# Tip: Using a slice when comparing
In a situation like this:
```python
a = Signal(unsigned(16))
b = Signal(unsigned(16))
c = Signal(unsigned(16))

m.d.comb += c.eq(a+b)
```
We expect that if `a+b` overflows, `c` will just be the lower 16 bits of the result.
```python
a = Signal(unsigned(16))
b = Signal(unsigned(16))
z = Signal()

m.d.comb += z.eq((a+b)== 0)
```
However, z will be a 17-bit signal. So a 16-bit overflow is not a 17-bit overflow, and this will result in comparison failure. Such as `a = 0xffff` and `b = 0x0001`, the addition will be `z = 0x10000`, which obviously not zero as we expected. \
Therefore, be careful to slice the result
```python
m.d.comb += z.eq((a+b)[:16] == 0)
```
Alternatively, just use an intermediate signal (not recommanded):
```python
tmp = Signal(unsigned(16))

m.d.comb += tmp.eq(a+b)
m.d.comb += z.eq(tmp == 0)
```
This becomes especially insidious when combining unsigned and signed signals:
```python
ptr = Signal(unsigned(16))
addr = Signal(unsigned(16))
offset = Signal(signed(5))

m.d.comb += ptr.eq(addr + offset)
```
we expect `ptr` to be a 16-bit value, since that is what we set it to be. However, what happens here?
Suppose `addr` is 0 and `offset` is -1. Will this comparison work? No, sorry my dear. It wouldn't work.
Consider that `adder` range from `0x7FFF to 0xFFFF` and `offset` range from `0x7 to 0xF`, which is max((+32767 ~ -32768) + (+7 ~ -8) )= `0x8006`
```python
y = Signal()
m.d.comb += y.eq((addr + offset) == 0xFFFF )
```
So the result of `addr+offset` in this is -1, which 2's complement 18-bit is `0x3FFFF`. If we slice it, it will be `0xFFFF`.
# Concatenating signals
You can create a new signal out of other signals using `Cat`:
```python
m.d.comb += x.eq(Cat(a, b, ...))
```
This concatenates the given signals *first element last* This is important that a in the example above ends up as the least significant bits of x. That is, the concatenation of `a` and `b` is not `ab` but `ba`. \
It is now easy to swap the bytes of a 16-bit signal:
```python
m.d.sync += x.eq(Cat(x[8:], x[:8]))
```
You can also assign to a `Cat`, so swapping the bytes can be accomplished in this way also:
```python
m.d.sync += Cat(x[8:], x[:8]).eq(x)
```
# Replicating signals
You can replicate a signal by concatenating it to itself via `Cat(x,x)`. But you can also replicate the signal via `Repl(x,2)`
`Repl` with `Cat` can be used together to, for example, signed-extend a value:
```python
uint16 = Signal(unsigned(16)) 
int32 = Signal(signed(32))

m.d.comb += int32.eq(Cat(uint16, Repl(uint16[15],16)))
```
Of course, the same can be done by simply using the right signal types:
```python
uint16 = Signal(unsigned(16)) 
int32 = Signal(signed(32))

m.d.comb += int32.eq(uint16)
```
The generated cide will do the right thing.
# Arrays
You can create an array of signals like this:
```python
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
You can even create multidimensional arrays:
```python
# Creates a 3 by 5 array of 16-bit signals:
yy = Array([Array[Signal(unsigned(16)) for _ in range(5)] for _ in range(3) ])
```
You can index into the array with a constant:
```python
z = y[2]
```
This will result in an "elaborate time" error if the index is out of bounds.
However, you can also index with another signal:
```python
i = Signal(unsigned(16))
z = y[i]
```
Of course, during elaboration this will not result in any error. The actual result depends on runtime. It is best to ensure as much as possible that your access is not invalid. One way is to declare the index to only have a valid range.
```python
y = Array([Signal(unsigned(16)) for _ in range(5)])
i = Signal.range(5)

z = y[i]
```
Of course, there is nothing to prevent `i` from being 5, 6, or 7, since it is a 3-bit signal.
Another way is to simply deal with invalid values:
```python
y = Array([Signal(unsigned(16)) for _ in range(5)])
i = Signal.range(5)
z = y[i % 4]
```
Note here, it will still result in unexpected result.
In the end, you will have to `formally verify` that i will only contain valid values.

# Records
A `Record` is a bundle of signals. To define a `Record`, we first must define a `Layout`.
## Layouts
```python
from nmigen.hdl.rec import *

class MyLayout(Layout):
    def __init__(self):
        super().__init__([
            (<signal_name>, <shape|layout> [, <direction>]),
            (<signal_name>, <shape|layout> [, <direction>]),
        ])
```
Here is an example of a bus with 8-bit data, 16-bit address line, and some control signals:
```python
class BusLayout(Layout):
    def __init__(self):
        super().__init__([
            ("data", unsigned(8)),
            ("addr", unsigned(16)),
            ("wr",1),
            ("en",1),
        ])
```
If your bus is very complex and easy to reuse another bus, you can define a bus by another bus.
```python
class DataBusLayout(Layout):
    def __init__(self):
        super().__init__([
            ("data",unsigned(8))
        ])
class AddrBusLayout(Layout):
    def __init__(self):
        super().__init__([
            ("addr",unsigned(16))
        ])
class AllBusLayout(Layout):
    def __init__(self):
        super().__init__([
            ("addr_bus",AddrBusLayout()),
            ("data_bus",DataBusLayout()),
        ])
```
## Laying out a record with a layout
Once a `Layout` is defined, you can define a `Record` using that `Layout`, and use it as a signal:
```python
class Bus(Record):
    def __init__(self):
        super().__init__(BusLayout())
...
# Later in a module:
    self.bus = Bus()
    m.d.comb += self.bus.data.eq(0xFF)
    m.d.sync += self.bus.wr.eq(0)
# Can assign a bus by a bus

    self.bus2 = Bus()
    m.d.comb += self.bus2.eq(self.bus)
```
# Directions and connecting records
It is often advantageous to define signals so that the zero value means either invalid or inactive. That way, you can have many of those signals and logical-or them together. For example, you might have three modules, each of which output a one-bit write signal, but only one module will write at a time. Then if your write signal is active high( zeros means no write), you can simply logical-or the write signal from each module together to get a master write signal.
![Imgur](https://i.imgur.com/114qlux.png)
```python
self.master_record = Bus()
m.d.comb += self.master_record.connect(bus1, bus2, bus3, ...)
```
The `connect` method on a record returns an array of statement which logical-ors each signal together.
The exactly same thing could be accomplished "manually".
```python
self.master_record = Bus()
m.d.comb += self.master_record.eq(bus1 | bus2 | bus3 |...)
```
The disadvantage is that `connect` can connect *parts* of records, if the field names matched. In this sense, the "subordinate" records must have every signal that the "master" record has. That is, the "subordinate" records can have extra signals, but the "master" record must not. \
Fan-out is where each subordinate record gets a copy of the master record. If the direction of each signal in the layout of record is `DIR_FANOUT`, then you can connect several records to a "master" record like this:
```python
self.master_record = Bus()
m.d.comb += self.master_record.connect(bus1, bus2, bus3, ...)
```
The syntax is exactly the same, but the direction is different, from master record to each subordinate record. Again, you could do this "manually":
```python
self.master_record = Bus()
m.d.comb += [
    bus1.eq(self.master_record),
    bus2.eq(self.master_record),
    bus3.eq(self.master_record),
    ...
]
```
But this is longer, and also doesn't handle when the master record has extra signals that not in the subordinate records.