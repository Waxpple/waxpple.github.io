---
title: "[Day07] Synthesis"
date: 2021-07-18T11:20:21+08:00
draft: false
---
# Supported devices
See the [vendor directory] (https://github.com/m-labs/nmigen/tree/master/nmigen/vendor) for supported devices and toolchain details.

Devices supported as of 18 JUL 2021:

|Device | Platform | Toolchain required|
|-------|----------|-------------------|
|Lattice iCE40||Yosys+nextpnr, LSE-iCECube2, Synplify-iCECube2|
|Lattice MachXO2||Diamond|
|Lattice ECP5||Yosys+nextpnr, Diamond|
|Xilinx Spartan 3A|Xilinx | ISE|
|Xilinx Spartan 6|Xilinx | ISE|
|Xilinx 7-series (Arty, Spartan, Kintex, Virtex)|Xilinx|Vivado|
|Xilinx UltraScale|XilinxUltraScalePlatform|Vivado|
|Intel|IntelPlatform|Quartus|
# Defining your board
Many boards are defined for you at [nmigen_boards](https://github.com/m-labs/nmigen-boards/tree/master/nmigen_boards).

You can copy one from there and modify it to suit you needs, or create a new class subclassed from one of the above supported device platform classes.
For example, I am using [Xilinx-KC705](https://github.com/m-labs/nmigen-boards/blob/master/nmigen_boards/kc705.py) for my development.
## Class properties
- `device`: a string. See the base platform class for which one to choose. This affects options passed to the toolchain so that it compiles for the correct chip.
- `package`: a string. See the base platform class for which one to choose. This affects options passed to the toolchain so that it compiles for the correct package of the chip.
- `resources`: a list of `Resource`. This names the pins you want to use, and configuration options for each such pin.
- `default_clk`: the name of the resource that is the clock for the default clock domain.
- `default_rst`: the name of the resource that is the reset for the default clock domain.
- `connectors`: optional, a list of `Connector`. It isn't obvious what purpose this serves. It may have something to do with certain toolchains.
## Resources
A `Resource` is a structure that contains a name, a number, and one or more configuration items for the resource. Adding a `Resource` to a board does two things:
- configures pin on the device
- allows you to request a resource's `pin` by name from the platform in your `elaborate` function.Such a `pin` has several `signal` associated with it, such as `i` for input and `o` for output, which you can then use in your module. 

For example, by including this `Resource` into the platform's `resource` list:
```
Resource("abc", 0, Pins("J3", dir="i"))
```
Then pin `J3` on the device will be configured as an input, and you can request the `abc` resource's input `Signal` like this
```
platform.request("abc").i
```
## Resource configuration items
- `Pins`: specifies the space-separated pin names associated with resource, their direction type, and whether the signal should be automativally inverted when crossing the pin(for, e.g., active low signals). Direction types are:
>- `i`: input only. Signal for the pin is `.i`
>- `o`: output only. Signal for the pin is `.o`
>- `io`: bidirectional. Signals for the pin are `.o` for the output, and `.oe` is the direction for the pin: 0 for input, 1 for output.
>- `oe`: tristate. Signals for the pin `.o` for the output, and `.oe` to enable output: 0 for disable, 1 for enable.
- `PinsN`: shorthand for `Pins`, where all pins are active low.
- `DiffPairs`: specifies the space-separated pin names for one or more differential pairs (positive and negative pins)
- `Clock`: specifies that the resource is a clock with the given frequency in Hz.
- `Attrs`: platform-specific attributes such as voltage standard to select.

A full resource specification for a clock pin used on a Lattice ICE40 board could be as follows:
```
Resource("clk", 0, Pins("J3",dir="i"), Clock(12e6),Attrs(GLOBAL=True, IO_STANDARD="SB_LVCMOS"))
```
This example says that the `clk` resource is at pin `J3` on the FPGA. `J3` is defined by vender of your FPGA. For example, KC705 pin map can be found at [here](https://www.xilinx.com/support/documentation/boards_and_kits/kc705/ug810_KC705_Eval_Bd.pdf). `clk` has a frequency of 12MHz, is a "global" signal, and uses the LVCMOS voltage standard. Without knowing about the toolchain for the platform, you will not know what attributes are required.
## Example for the Xilinx-KC705.
```
import os
import subprocess

from nmigen.build import *
from nmigen.vendor.xilinx_7series import *
from .resources import *


__all__ = ["KC705Platform"]


class KC705Platform(Xilinx7SeriesPlatform):
    device      = "xc7k325t"
    package     = "ffg900"
    speed       = "2"
    default_clk = "clk156"
    resources   = [
        Resource("clk156", 0, DiffPairs("K28", "K29", dir="i"),
                 Clock(156e6), Attrs(IOSTANDARD="LVDS_25")),

        *LEDResources(pins="AB8 AA8 AC9 AB9 AE26 G19 E18 F16",
                      attrs=Attrs(IOSTANDARD="LVCMOS15")),

        UARTResource(0,
            rx="M19", tx="K24",
            attrs=Attrs(IOSTANDARD="LVCMOS33")
        ),
    ]
    connectors  = []

    def toolchain_program(self, products, name):
        openocd = os.environ.get("OPENOCD", "openocd")
        with products.extract("{}.bit".format(name)) as bitstream_filename:
            subprocess.check_call([openocd,
                "-c", "source [find board/kc705.cfg]; init; pld load 0 {}; exit"
                      .format(bitstream_filename)
            ])


if __name__ == "__main__":
    from .test.blinky import *
    KC705Platform().build(Blinky(), do_program=True)
```
You can see that it is not well define yet. Let's take a look into arty-a7 board.
```
import os
import subprocess

from nmigen.build import *
from nmigen.vendor.xilinx_7series import *
from .resources import *


__all__ = ["ArtyA7Platform"]


class ArtyA7Platform(Xilinx7SeriesPlatform):
    device      = "xc7a35ti"
    package     = "csg324"
    speed       = "1L"
    default_clk = "clk100"
    resources   = [
        Resource("clk100", 0, Pins("E3", dir="i"),
                 Clock(100e6), Attrs(IOSTANDARD="LVCMOS33")),

        *LEDResources(pins="H5 J5 T9 T10", attrs=Attrs(IOSTANDARD="LVCMOS33")),

        RGBLEDResource(0, r="G6", g="F6", b="E1", attrs=Attrs(IOSTANDARD="LVCMOS33")),
        RGBLEDResource(1, r="G3", g="J4", b="G4", attrs=Attrs(IOSTANDARD="LVCMOS33")),
        RGBLEDResource(2, r="J3", g="J2", b="H4", attrs=Attrs(IOSTANDARD="LVCMOS33")),
        RGBLEDResource(3, r="K1", g="H6", b="K2", attrs=Attrs(IOSTANDARD="LVCMOS33")),

        *ButtonResources(pins="D9  C9  B9  B8 ", attrs=Attrs(IOSTANDARD="LVCMOS33")),
        *SwitchResources(pins="A8  C11 C10 A10", attrs=Attrs(IOSTANDARD="LVCMOS33")),

        UARTResource(0,
            rx="A9", tx="D10",
            attrs=Attrs(IOSTANDARD="LVCMOS33")
        ),

        Resource("cpu_reset", 0, Pins("C2", dir="o"), Attrs(IOSTANDARD="LVCMOS33")),

        SPIResource(0,
            cs="C1", clk="F1", mosi="H1", miso="G1",
            attrs=Attrs(IOSTANDARD="LVCMOS33")
        ),

        Resource("i2c", 0,
            Subsignal("scl",        Pins("L18", dir="io")),
            Subsignal("sda",        Pins("M18", dir="io")),
            Subsignal("scl_pullup", Pins("A14", dir="o")),
            Subsignal("sda_pullup", Pins("A13", dir="o")),
            Attrs(IOSTANDARD="LVCMOS33")
        ),

        *SPIFlashResources(0,
            cs="L13", clk="L16", mosi="K17", miso="K18", wp="L14", hold="M14",
            attrs=Attrs(IOSTANDARD="LVCMOS33")
        ),

        Resource("ddr3", 0,
            Subsignal("rst",    PinsN("K6", dir="o")),
            Subsignal("clk",    DiffPairs("U9", "V9", dir="o"), Attrs(IOSTANDARD="DIFF_SSTL135")),
            Subsignal("clk_en", Pins("N5", dir="o")),
            Subsignal("cs",     PinsN("U8", dir="o")),
            Subsignal("we",     PinsN("P5", dir="o")),
            Subsignal("ras",    PinsN("P3", dir="o")),
            Subsignal("cas",    PinsN("M4", dir="o")),
            Subsignal("a",      Pins("R2 M6 N4 T1 N6 R7 V6 U7 R8 V7 R6 U6 T6 T8", dir="o")),
            Subsignal("ba",     Pins("R1 P4 P2", dir="o")),
            Subsignal("dqs",    DiffPairs("N2 U2", "N1 V2", dir="io"),
                                Attrs(IOSTANDARD="DIFF_SSTL135")),
            Subsignal("dq",     Pins("K5 L3 K3 L6 M3 M1 L4 M2 V4 T5 U4 V5 V1 T3 U3 R3", dir="io"),
                                Attrs(IN_TERM="UNTUNED_SPLIT_40")),
            Subsignal("dm",     Pins("L1 U1", dir="o")),
            Subsignal("odt",    Pins("R5", dir="o")),
            Attrs(IOSTANDARD="SSTL135", SLEW="FAST"),
        ),

        Resource("eth_clk25", 0, Pins("G18", dir="o"),
                 Clock(25e6), Attrs(IOSTANDARD="LVCMOS33")),
        Resource("eth_clk50", 0, Pins("G18", dir="o"),
                 Clock(50e6), Attrs(IOSTANDARD="LVCMOS33")),
        Resource("eth_mii", 0,
            Subsignal("rst",     PinsN("C16", dir="o")),
            Subsignal("mdio",    Pins("K13", dir="io")),
            Subsignal("mdc",     Pins("F16", dir="o")),
            Subsignal("tx_clk",  Pins("H16", dir="i")),
            Subsignal("tx_en",   Pins("H15", dir="o")),
            Subsignal("tx_data", Pins("H14 J14 J13 H17", dir="o")),
            Subsignal("rx_clk",  Pins("F15", dir="i")),
            Subsignal("rx_dv",   Pins("G16", dir="i"), Attrs(PULLDOWN="TRUE")), # strap to select MII
            Subsignal("rx_er",   Pins("C17", dir="i")),
            Subsignal("rx_data", Pins("D18 E17 E18 G17", dir="i")),
            Subsignal("col",     Pins("D17", dir="i")),
            Subsignal("crs",     Pins("G14", dir="i")),
            Attrs(IOSTANDARD="LVCMOS33")
        ),
        Resource("eth_rmii", 0,
            Subsignal("rst",       PinsN("C16", dir="o")),
            Subsignal("mdio",      Pins("K13", dir="io")),
            Subsignal("mdc",       Pins("F16", dir="o")),
            Subsignal("tx_en",     Pins("H15", dir="o")),
            Subsignal("tx_data",   Pins("H14 J14", dir="o")),
            Subsignal("rx_crs_dv", Pins("G14", dir="i")),
            Subsignal("rx_dv",     Pins("G16", dir="i"), Attrs(PULLUP="TRUE")), # strap to select RMII
            Subsignal("rx_er",     Pins("C17", dir="i")),
            Subsignal("rx_data",   Pins("D18 E17", dir="i")),
            Attrs(IOSTANDARD="LVCMOS33")
        )
    ]
    connectors  = [
        Connector("pmod", 0, "G13 B11 A11 D12 - - D13 B18 A18 K16 - -"), # JA
        Connector("pmod", 1, "E15 E16 D15 C15 - - J17 J18 K15 J15 - -"), # JB
        Connector("pmod", 2, "U12 V12 V10 V11 - - U14 V14 T13 U13 - -"), # JC
        Connector("pmod", 3, " D4  D3  F4  F3 - -  E2  D2  H2  G2 - -"), # JD

        Connector("ck_io", 0, {
            # Outer Digital Header
            "io0":  "V15",
            "io1":  "U16",
            "io2":  "P14",
            "io3":  "T11",
            "io4":  "R12",
            "io5":  "T14",
            "io6":  "T15",
            "io7":  "T16",
            "io8":  "N15",
            "io9":  "M16",
            "io10": "V17",
            "io11": "U18",
            "io12": "R17",
            "io13": "P17",

            # Inner Digital Header
            "io26": "U11",
            "io27": "V16",
            "io28": "M13",
            "io29": "R10",
            "io30": "R11",
            "io31": "R13",
            "io32": "R15",
            "io33": "P15",
            "io34": "R16",
            "io35": "N16",
            "io36": "N14",
            "io37": "U17",
            "io38": "T18",
            "io39": "R18",
            "io40": "P18",
            "io41": "N17",

            # Outer Analog Header as Digital IO
            "a0": "F5",
            "a1": "D8",
            "a2": "C7",
            "a3": "E7",
            "a4": "D7",
            "a5": "D5",

            # Inner Analog Header as Digital IO
            "io20": "B7",
            "io21": "B6",
            "io22": "E6",
            "io23": "E5",
            "io24": "A4",
            "io25": "A3"
        }),
        Connector("xadc", 0, {
            # Outer Analog Header
            "vaux4_n":   "C5",
            "vaux4_p":   "C6",
            "vaux5_n":   "A5",
            "vaux5_p":   "A6",
            "vaux6_n":   "B4",
            "vaux6_p":   "C4",
            "vaux7_n":   "A1",
            "vaux7_p":   "B1",
            "vaux15_n":  "B2",
            "vaux15_p":  "B3",
            "vaux0_n":  "C14",
            "vaux0_p":  "D14",

            # Inner Analog Header
            "vaux12_n": "B7",
            "vaux12_p": "B6",
            "vaux13_n": "E6",
            "vaux13_p": "E5",
            "vaux14_n": "A4",
            "vaux14_p": "A3",

            # Power Measurements
            "vsnsuv_n":   "B17",
            "vsnsuv_p":   "B16",
            "vsns5v0_n":  "B12",
            "vsns5v0_p":  "C12",
            "isns5v0_n":  "F14",
            "isns5v0_n":  "F13",
            "isns0v95_n": "A16",
            "isns0v95_n": "A15",
        })
    ]

    def toolchain_prepare(self, fragment, name, **kwargs):
        overrides = {
            "script_before_bitstream":
                "set_property BITSTREAM.CONFIG.SPI_BUSWIDTH 4 [current_design]",
            "script_after_bitstream":
                "write_cfgmem -force -format bin -interface spix4 -size 16 "
                "-loadbit \"up 0x0 {name}.bit\" -file {name}.bin".format(name=name),
            "add_constraints":
                "set_property INTERNAL_VREF 0.675 [get_iobanks 34]"
        }
        return super().toolchain_prepare(fragment, name, **overrides, **kwargs)

    def toolchain_program(self, products, name):
        xc3sprog = os.environ.get("XC3SPROG", "xc3sprog")
        with products.extract("{}.bit".format(name)) as bitstream_filename:
            subprocess.run([xc3sprog, "-c", "nexys4", bitstream_filename], check=True)


if __name__ == "__main__":
    from .test.blinky import *
    ArtyA7Platform().build(Blinky(), do_program=True)
{"mode":"full","isActive":false}
```

# Building
```
python3 file.py
```
This will result in a directory, `build`, containing the output files:
- `top.il`: The ilang output for yosys.
- `top.bin`: The bitstream to send to the device (e.g. via iceprog)
- `top.rpt`: Statistics from nextpnr. The most useful is the cell and LUT
- `top.tim`: Timing analysis. Shows how fast you can go. Probably.

