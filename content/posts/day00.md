---
title: "[Day00] nMigen 戰鬥"
date: 2021-07-09T14:33:00+08:00
draft: false
---
# 第一天接觸nMigen, 先來安裝　nMigen!
![1](https://m-labs.hk/images/migen@2x.png?h=43d4e86170d805ce58f90901ae31a64141ea32606f7cefcb4b2b165e362c2b9a)

# 安裝nMigen
```
pip install git+https://github.com/m-labs/nmigen.git
pip install git+https://github.com/m-labs/nmigen-boards.git
```
安裝之後就可以開始學習如何使用nMigen製作電路
# Value in migen
## Const 永不變
```
from nmigen import *
a = Const(10)
a.shape()
>> Shape(width=4, signed=False)
a = Const(10)
a.shape()
>> Shape(width=5, signed=True)
x = Const(3,range(-5,11))
x.shape()
>> Shape(width=5, signed=True)
```
## 可以使用Enum 來做常數狀態
```
from enum import Enum, unique

@unique
class Func(Enum):
    NONE = 0
    ADD = 1
    SUB = 2
    MUL = 3
    DIV = 4

...

>>> x = Const(2, Func)
>>> x.shape()
unsigned(3)

>>> x = Value.cast(Func.NONE)
>>> x
const 3'd0

```
## Signal 是 Wire或者reg

```
>>> from nmigen import *
>>> A = Signal(signed(8))
>>> A.shape()
Shape(width=8, signed=True)
```
```
>>> x = Signal(range(-5,11))
>>> x.shape()
Shape(width=5, signed=True)
```
```
>>> x = Signal(Func)
>>> x
(sig x)
>>> x = Signal(unsigned(16),name="dout")
>>> x.name
'dout'
```

---
>但是我安裝Miniconda 後整個WSL的ssl都掛了，建議不要安裝
># 安裝Miniconda
>因為我是用WSL，也沒有GUI，就直接用miniconda就好了。
>```
>wget https://repo.anaconda.com/miniconda/Miniconda3-py39_4.9.2-Linux-x86_64.sh
>chmod +x Miniconda3-py39_4.9.2-Linux-x86_64.sh
>./Miniconda3-py39_4.9.2-Linux-x86_64.sh
>```
