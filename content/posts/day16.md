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
因此之後幾個篇文章都會跟隨這本書，雖然這本書裡面是寫簡體字，用語跟台灣相當不一樣xD 但是原作者就是中國人且經歷相當豐富，希望看完這本書能收穫滿滿。

本書是介紹 蜂鳥E200系列的處理器，本書開源的是面相低功耗低面積的嵌入式處理器，依照作者給的數據應該是全勝於ARM Cortex-M0/M0+ 系列能耗比。
開源網址在早期，也就是這本書剛出版的時候為 [GitHub網址](https://github.com/SI-RISCV/e200_opensource) 

但是原網址已經不維護與更新了，最新版本在[新版本GitHub網址](https://github.com/riscv-mcu/e203_hbirdv2)。讀者可以自行前往比對，我應該是先以舊版的搭配書嗑完後再去run看看新版的。