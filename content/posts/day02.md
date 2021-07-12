---
title: "[Day02] Basic terminology"
date: 2021-07-12T10:01:41+08:00
draft: false
---

# Domains
A *domain* in its basic definition, is a grouping of logic cells. If consider a module as a blackbox. With its inputs and outputs, any given output is generated within one and only one domain. \
`Modules` come with two domain built in: a combinational domain and a sequential(synchronous) domain.
# Combinational
Logic that contains no clocked elements is called combinational logic. This is one of the domain that a `Module` contains. It is always named `comb`, and it can be accessed via m.d.comb. \
m means `Module` and d means `domain`.
# Synchronous
Logic that contains clocked elements is called synchronous (a.k.a sequential circuit) because all the FFs are triggered by particular clock domain. Each clock domain also has a reset signal which can reset all FFs to a given state. Finally, the domain specifies the edge of its clock on which all the FFs change: `posedge` or `negedge`. \
Unless otherwise specified, there is one synchronous domain in a `Module` called sync. It can be accessed via `m.d.sync`.
# Creating more domains
There is no reason to create combinational domains. As mentioned above, modules already contain one combinational domain, comb. \
You can create a synchronous clock domain using `ClockDomain("<domain-name>", clk_edge="pos|neg")`. This gives you both the clock and the reset signal for the domain. By default, the domain name is `sync` and the clock edge is `pos`.
```
m = Module()
mydomain = ClockDomain("clk")
m.domains += mydomain

m.d.mydomain += ... # logic to add in the "mydomain" clock domain.
```
You can access a domain within a module by its name. So a domain created via ClockDomain("my clock") is accessed via `m.d.myclock` or `m.d["myclk"]`. \
- `ClockSignal(domain="<domain>")` gives you the clock signal for the given domain.
- `ResetSignal(domain="<domain>")` gives you the reset signal for the given domain.
# Tips: Clock domains with the same clock but different edges.
This can be done simply by creating one `clockdomain`
```
pos = ClockDomain("pos")
neg = ClockDomain("neg", clk_edge="neg")
```
Next, assign them with same clock driven
```
neg.clk = pos.clk
neg.rst = pos.rst
```
And then you can add these to the module. We can add more than domain to a module with the same statement.
```
m.domains += [pos, neg]
```
# Access to domains
A module can access its domains via its `d` attribute. By default, if a synchronous domain is added to a module's `domain`, then all modules everywhere will also have access to that domain via their d attribute, even if that module is not a submodule of the module where the domain was added.
```
m = Module()
m2 = Module()
m.domains += ClockDomain("thing")
m.d.thing += # logic
# This is implicity 
m2.d.thing += #logic
```
If you want to explicitly inhibit this global propagation by setting the `local` named parameter of the `ClockDomain` to `True`.
```
m = Module()
m2 = Module()
m.domains += ClockDomain("thing", local=True)
m.d.thing += # logic
# Error will occur here.
m2.d.thing += # logic
```
# Ports
The equivalent of ports in a module is public attributes. In the following example, `a` and `data` are publicly available to other modules, while `b` is not, just as `a` and `data` are publicly available to other python classes, and `b` is not.
```
class ThingBlock(Elaboratable):
    def __init__(self):
        # Public accessible
        self.a = Signal()
        self.data = Signal(8)
    def elaborate(self, platform: str):
        m = Module()
        # Internal use reg/wire.
        b = Signal()
        return m
```
# Reset/default values for signals
If a `Signal` is set in the *combinational* or *synchronous* domain, then you can specify the default value of the signal if it is not set. By default value of the signal if it is not set. By default, it is zero, but for a non-zero value, you can specify the default value for a signal when constructing the signal by setting the `reset` named parameter in the constructor. For example, this create a 16-bit signed signal, self.x, which initial value set to 0xFFFF.
```
self.x = Signal(signed(16), reset=0xFFFF) # initial value "0xFFFF"
```
# Explicitly not resetting
For synchronous signals(that is, a signal set in a synchronous domain), you can specify that it is not reset on the reset signal, instead only getting an initial balue on power-up. This is done by setting the `reset_less` named parameter in the constructor to `True`
```
self.x = Signal(signed(16),reset= 0xFFFF, reset_less = True)
```
This is especially useful during simulation or formal verification where you want to activate the reset, but keep some signals "outside" the reset. For example, a cycle counter that maintains its count across resets.