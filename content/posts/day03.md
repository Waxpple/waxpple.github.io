---
title: "[Day03] Branching"
date: 2021-07-12T14:38:07+08:00
draft: false
---
# If-Elif-Else
you cannot use the standard python `if-elif-else` statements to create statements. Instead, using nMigen branching.
```
with m.If(condition1):
    m.d.comb += statements1
with m.Elif(condition2):
    m.d.comb += statements2
with m.Else():
    m.d.comb += statements3
```
If you use regular Python `if-elif-else`, then those will be evaluated during *generation* of the logic, not the logic itself. This can be useful if you want a flag to cause different logic implement to be generated, and this is a good use of platform string pass to elaborate().
```
if (platform== "KC705")
    m.d.comb += statement1
else:
    m.d.comb += statement2
```
If `platform` is `"KC705"` then statement1 will be implement in generated hardware, otherwise only statement2 will be implement in the design.

# Conditions
The conditions in `If-Elif-Else` are cimparisons, for example `a == 1` or `(a >= b) & (a <= c)`. Note that each comparison will be one-bit comparison. \
If you have a signal with more than one bit and use it as the condition, use `with m.If(a):`, then the condition will be true if any bit in a is 1.

# Switch-Case-Default
You can use `Switch-Case-Default` just as in standard HDLs using the following `with` constructs:
```
with m.Switch(expression):
    with m.Case(value1):
        statements1
    with m.Case(value2):
        statements2
    with m.Default():
        statements3
```
Although it is suggested that using `full-case` switch that we used to do in verilog. You can use multiple values in one case statement.
```
with m.Switch(expression):
    with m.Case(value1,value2):
        statements1
    with m.Default():
        statements3
```
You can leave out the `Defalt()`, but not suggest you to do that.
```
m.d.comb += x.eq(1)
with m.Switch (y):
    with m.Case(0,1,2):
        m.d.comb += x.eq(2)
```
In this example, if `y is in [0, 1, 2]` then `x` is assigned 2. Otherwise `x` retains its value of 1.
- Recall the section on overriding statements. One signal can only assigned in only one clock domain.

# Specify a bit patterns
The way to specify a matching pattern in a `Case` is with a Python string of binary digits. For example, `"0011101011"`. A don't-care bit is specified using a dash `-`, so for example `"00111-----"`. The number of bits in the string must exactly the same as the number of bits in the expression it is being compared to.
```
with m.If(a.matches("11---",3,b)):
    statement1
```
Is equivelent to
```
with m.Switch(a):
    with m.Case("11---",3,b):
        statement1
```