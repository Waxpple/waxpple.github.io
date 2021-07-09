---
title: "[Day00] nMigen 戰鬥"
date: 2021-07-09T14:33:00+08:00
draft: false
---
第一天接觸nMigen, 先來安裝　nMigen!

>但是我安裝Miniconda 後整個WSL的ssl都掛了，建議不要安裝
># 安裝Miniconda
>因為我是用WSL，也沒有GUI，就直接用miniconda就好了。
>```
>wget https://repo.anaconda.com/miniconda/Miniconda3-py39_4.9.2-Linux-x86_64.sh
>chmod +x Miniconda3-py39_4.9.2-Linux-x86_64.sh
>./Miniconda3-py39_4.9.2-Linux-x86_64.sh
>```

```
pip install git+https://github.com/m-labs/nmigen.git
pip install git+https://github.com/m-labs/nmigen-boards.git
```
