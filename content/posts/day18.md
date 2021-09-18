---
title: "[Day01] 有時候學硬體或者設計硬體累了...來點軟體吧!"
date: 2021-09-18T09:57:09+08:00
draft: false
---

# 前言

雖然我正在Mediatek實習，待的部門是做手機CPU的部門，不是RTL designer 是做 Implement team的。其實心中預期比較多的是實習能碰Implement 因為在學校其實沒甚麼機會走到PnR, Sign-off等等後端真正的流程。大多是RTL打一打，丟進工具與幾個library隨便合成後跑跑Netlist sim就完工了。一部分是因為走完這些的時間就不太夠了更不太可能在學校上課的期間能走到GDS Tapout。但我發現每天看standard cell、每天看數位電路其實也蠻膩的。於是大膽下定決心累的時候除了運動，可以多練習軟實力。如果以後數位設計失業至少還能寫code維生xD

# 軟體要來學點甚麼?

![meme](https://aws1.discourse-cdn.com/codecademy/original/5X/b/f/2/a/bf2a43e0d76918d1f38537acc9b59d6c245a6eb1.jpeg)

我之前聽了一個CS部門的同事分享，雖然資料結構、作業系統和演算法甚麼得很重要，但說真的他最受用無窮的就是軟體設計模式(software design pattern)。那時候在台下的我覺得很有道理，Design pattern似乎是一個放諸四海皆準的一個主題，可以幫助你在專案中的coding style等更一致更有準則，容易解釋容易讓人參與其中。所以我就決定在設計Risc v的閒餘時間多學習設計模式。當然這也算是side project另外一個Side project xD

![meme](https://i.pinimg.com/originals/1d/27/6b/1d276bed4d5c812996db8114ea7b4a93.jpg) 

目前就陳列一下side project:

1. LiteX on iCE-sugarpro
2. E203 Risc-V re-implement combine opensource skywater 130nm pdk(?)或許可以搭Google的車Tapeout，等學到一定程度就到openIC study group上號召XD
3. Software design pattern

廢話不多說，我是在網路上找到一個IT鐵人賽，教你學設計模式　[[ Day 1 ] 我為什麼想學設計模式 ( Design Pattern )](https://ithelp.ithome.com.tw/articles/10201706) 但是我應該會想用C/C++來寫這個課題，而這位大神是使用Java。

