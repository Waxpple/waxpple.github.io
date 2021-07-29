---
title: "[Day12] Simulating memory on a Wishbone Bus"
date: 2021-07-28T21:06:11+08:00
draft: false
---

# Reading from memory

This post follows tutorial on [https://vivonomicon.com/2020/04/14/learning-fpga-design-with-nmigen/](https://vivonomicon.com/2020/04/14/learning-fpga-design-with-nmigen/)

On the iCESugar-pro, there is a `Winbond w25q256jv` which is a 3V 256M-bit serial flash memory with dual/quad spi. Here is a document [https://github.com/wuxx/icesugar-pro/blob/master/doc/w25q256jv.pdf](https://github.com/wuxx/icesugar-pro/blob/master/doc/w25q256jv.pdf) of the SPI module. Using this module, we can program the FPGA by writing the bitstream into flash memory. 
To test reading data from the flash chip, we can write a new module which contains a simple state machine to read data, and a parent module which acts like a very simple CPU. The parent module will increment the address that it reads from, and depending on the returned value, it will either delay for a given number of cycles, toggle one of the leds, or jump back to address zero. This will let us write simple "programs" which set the leds to different colors at different times.

## Simulating memory on a wishbone bus

Before writing the actual SPI flash module, let's write and test the parent "CPU" module with simulated memory values. The SPI interface has a few "gotchas" and places where incorrect timing can cause problems, so it is a good idea to make sure that the basic program logic works before diving into real case.

The CPU module needs to be able to request new reads and wait for the memory module to finish, without knowing how long a memory access will take to be ready. This is a common problem which people have already developed standards for, so I an going to use the wishbone bus standard to handle data transfer. nMigen includes an implementation of this sort of bus in the `nmigen-soc`.

# Install nMigen-SoC

```
git clone https://github.com/nmigen/nmigen-soc
cd nmigen-soc
sudo python3 setup.py install
```

# Building memory module in nMigen

To use memory module from nmigen, we can make our memory module inherit from `nmigen_soc.wishbone.Interface`, call the parent `__init__` method with the desired address and data widths during initialization, and then access the bus signals with `self.<signal>`.

- `ack`: acknowledge signal which is asserted by the slave when it is finished with a transaction.
- `cyc`: cycle signal which is asserted by the master when a bus transaction is ongoing. The slave should ignore any inputs and avoid asserting any outputs when cycle is not asseted.
- `stb`: strobe signal which is asserted by the master when a bus transfer cycle is on flight. The strobe signal can be toggles multiple times while the cycle ginal is asserted to perform multiple transfers in a single transaction.
- `dat_r`: read data buffer which the slave fills with data to be read by the master once `ack` is asserted.

# Example on Simulating ROM

## ROM16.py

```python
from nmigen import *
from math import ceil, log2
from nmigen_soc.memory import *
from nmigen_soc.wishbone import *

class ROM( Elaboratable, Interface): 
    def __init__(self, data):
        self.size = len(data)
        self.data = Memory(width=32, depth= self.size, init=data)
        self.r = self.data.read_port()

        Interface.__init__( self, 
                            data_width = 32,
                            addr_width = ceil( log2(self.size +1))
                            )
        self.memory_map = MemoryMap(    data_width = self.data_width,
                                        addr_width = self.addr_width,
                                        alignment = 0
                                    )
    def elaborate( self, platform):
        m = Module()
        m.submodules.r = self.r
        m.d.sync += self.ack.eq(0)
        with m.If(self.cyc):
            m.d.sync += self.ack.eq(self.stb)
        m.d.comb += [
            self.r.addr.eq( self.adr),
            self.dat_r.eq(self.r.data)
        ]
        return m
```

## rom_read_ut.py

```python
from nmigen.back.pysim import *
from ROM16 import ROM
def rom_read_ut (rom, address, expected):
    yield rom.adr.eq(address)
    yield Tick()
    yield Settle()
    actual = yield rom.dat_r
    if expected == actual:
        print("PASS: Memory[0x%04X] = 0x%08X"%(address, expected))
    else:
        print("FAILED: Memory[0x%04X] = 0x%08X (got: 0x%08X)"%(address, expected, actual))

if __name__ == "__main__":
    dut = ROM( [ 0x01234567, 0x89ABCDEF, 0x0C0FFEE0, 0xDEC0FFEE, 0xFEEBEEDE])
    sim = Simulator(dut)
    sim.add_clock(1e-6)
    
    def proc():
        yield from rom_read_ut(dut, 0, 0x01234567)
        yield from rom_read_ut(dut, 1, 0x89ABCDEF)
        yield from rom_read_ut(dut, 2, 0x0C0FFEE1)
        yield from rom_read_ut(dut, 3, 0xDEC0FFEE)
        yield from rom_read_ut(dut, 4, 0xFEEBEEDE)
    sim.add_sync_process(proc)
    with sim.write_vcd("test.vcd", "test.gtkw"):
        sim.run()
```

Note: `Memory[2] Should equal to 0x0C0FFEE0`. Here is a simple demonstrate if the simulation works.

## Simulation

```
python3 rom_read_ut

PASS: Memory[0x0000] = 0x01234567
PASS: Memory[0x0001] = 0x89ABCDEF
FAILED: Memory[0x0002] = 0x0C0FFEE1 (got: 0x0C0FFEE0)
PASS: Memory[0x0003] = 0xDEC0FFEE
PASS: Memory[0x0004] = 0xFEEBEEDE
```



![Imgur](https://i.imgur.com/1X1aC9r.png)