---
title: "[Day16] RISC open hardware design"
date: 2021-08-31T11:47:55+08:00
draft: false
---

# What is the difference between RISC and CISC?

CISC (Complex-Instrument-Set-Computer) is an instrument set that contains common instruments and uncommon instruments. For example, 80% of the common instruments we used in the arithmetic program is 20% of the total instrument set.

RISC (Reduced-Instrument-Set-Computer) only contains common instruments. It will use more than one simple instrument to finshed one complex instrument which takes more cycles to do the same job but keep the processor simple and clean.

# 第一天開始學RISC-V ISA

我就去書店買了這本書來看 ![手把手教你設計CPU-RISCV處理器](https://cf-assets2.tenlong.com.tw/products/images/000/119/652/original/51KU7JzDTlL.jpg?1527073409)
因此之後幾個篇文章都會跟隨這本書，雖然這本書裡面是寫簡體字，用語跟台灣相當不一樣xD 但是原作者就是中國人，感覺經歷也不錯，希望能收穫滿滿。