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

To use memory module from nmigen, we can make our memory module inherit from `nmigen_soc.wishbone.Interface`, call the parent `__init__` method with the desired address and data widths during initialization, and then access the bus ginals with `self.<signal>`.

