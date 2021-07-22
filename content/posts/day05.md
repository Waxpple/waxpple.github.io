---
title: "[Day05] Simulations"
date: 2021-07-14T15:28:58+08:00
draft: false
---

# Simulating
The best way to simulate a module is through nMigen's `Simulator`.
## Define you ports
Define a `ports` function in your module which returns an array of your module's ports:
```python
class YourModule(Elaboratable):
    ...
    def ports(self):
        return [self.youmodule.p1, self.yourmodule.p2, ...]
```
## Create a top-level module
Create a top-level module for your simulation:
```python
from nmigen import *
from nmigen.back.pysim import Simulator, Delay, Settle
from somewhere import YourModule

if __name__ == "__main__":
    m = Module()
    m.submodules.yourmodule = yourmodule = YourModule()

    sim = Simulator(m)

    def process():
        # To be defined
    sim.add_process(process) # or sim.add_sync_process(process), see below
    with sim.write_vcd("test.vcd", "test.gtkw", traces=yourmodule.ports()):
        sim.run()
```
> There is currently a bug in nMigen where inputs to your module are not output to the trace file. To get around this, for each such input, place this in your `main` before the `Simulator` construction:
>```python
>input1 = Signal()
>m.d.comb += yourmodule.input1.eq(input1)
>...
>sim = Simulator(m)
>```
>Inside your `process`, refer to this input as `input1`, not `yourmodule.input1`. This will force nMigen to include `input1` in the trace file.
## Define your clocks, if any
If you have clocks, add each clock after the `Simulator` constrction, giving the clock period in seconds. For example, a 100MHz clock for clock domain `fast_clock` and a nearly 166MHz clock for `faster clock`.
```python
...
sim = Simulator()
sim.add_clock(1e-8,domain="fast_clock")
sim.add_clock(6e-9,domain="faster_clock")
```
Leaving out `domain` will cause the clock period to be assigned to the default clock domain, `sync`.
## The process function
The `process` function is a Python generator that nMigen calls to see what to do next in the simulation. Since it is a generator, process must `yield` a statement to perform. For example:
```python
def process():
    yield x.eq(0)
    yield y.eq(0xFF)
```
The above would set `x` to 0 and `y` to 0xFF, with effectively no delay between them.
You can yield nMigen `Value`, which you can then use to do various comparisons.
```python
def process():
    yield x.eq(0)
    yield Delay(1e-8) # delay 10 nano seconds
    yie y.eq(0xFF)
    yield Settle() # force all combinational computation to happen.
    got = yield yourmodule.sum
    want = yield (x+y)[:8]
    if got != want:
        print(f"Error, result={got:02x},golden={want:02x})
```
In the above example, `x` will be set to 0, then there will be 10 ns delay, then `y` will be set to 0xFF, all combinational logic will be given a chance to settle, and finally `yourmodule.sum` and `(x+y)[:8]` will be evaluated, and if they are not equal, a diagnostic message is sent to the terminal output.

You can even have more that one process running parallel.
```python
def x_process():
    yield Delay(1e-6)
    yield x.eq(0)
    yield Settle()
def y_process():
    yield Delay(1.2e-6)
    yield y.eq(0xFF)
    yield Settle()
sim.add_process(x_process)
sim.add_process(y_process)
```
In the above example, `x` will be set to 0 at time 1 us, and `y` will be set to 0xFF at time 1.2 us.

**Warning**: driving the same signal from more than one process can lead to undefined behavior if both processes assign to the signal simultaneously.

## Non-synchronous processes
If you want to specify exactly when signals change based on time, then you can create a non-synchronous `process`. You must add such a process to the simulator via `add_process`:
```python
sim.add_process(process)
```
## Synchronous process
If you want to specify when signals change base on clock edges, then you can create a synchronous `process`. You can add such a process to the simulator via `add_sync_process`, specifying the clock domain it should be clocked from:
```python
sim.add_sync_process(process1, domain= "name1")
sim.add_sync_process(process2, domain= "name2")
```
If you `yield` with no value from a synchronous process, then the process will wait for the next clock edge. Note that for synchronous processes, one clock edge will occur before the process starts, so take that into account when you look at your traces.
It is also important to understand when statements are executed in relation to clock edges. They are always executed infinitesimally after the previous clock edge. Thus, in this example.
```python
def process():
    yield x.eq(0)   #step1
    yield           #step2
    yield x.eq(1)   #step3
    yield           #step4
```
There will be one clock edge that always takes place before the process runs. Then `x` is set to 0 (step 1). Then another clock edge happens(step 2). `x` is set to 1 infinitesimally after that clock edge (step 3). Then another clock signals that appear to change coincident with a clock edge actuvally change just after that clock edge. Outputs that change like that can be consider to have been "driven" by the clock edge.
## Passive and active processes
Processed may be passive or active. When an active process runs out of things to tell the simulator to do, it asks the simulator to finish. In effect, it controls the endpoint of the simulator. The simulation ends when all active processes are done. A *passive* process, on the other hand, doesn't ask the simulator to finish.

By default, processes added with `add_process` and `add_sync_process` are active. A process can change its mode using `yield Active()` or `yield Passive()`.

## Ending the simulation
As mentioned above, the simulation ends when all active processes are done. This is how `sim.run()` works.

However, you can instead use `sim.run_until()`, which lets you end the simulation at a particular time. The `run_passive` key is `False` by default, meaning that the simulation will also end if all active processes are done. This behavior can be changed by setting `run_passive` to `True`, in which case the simulation will only end once the specified time is reached. For example, the following will run the simulation for 100 microseconds and then stop, regardless of whether the active processes are done.
```python
with sim.write_vcd("test.vcd", "test.gtkw", traces=yourmodule.ports()):
    sim.run_until(100e-6, run_passive=True)
```
## Running the simulation and viewing the output
The simulation is run simply by running the main module:
```python
python3 main_module.py
```
The output should be a `test.vcd` file and a `test.gtkw` file. Running `gtkwave` will allow you to view the output. Running it on `test.vcd` will make you select the signals you want to see when `gtkwave` opens, while running it on `test.gtkw` will open `gtkwave` showing the signals in the `traces` key that you you gave in the call to `sim.write_vcd()`.
```python
gtkwave test.vcd
gtkwave test.gtkw
```