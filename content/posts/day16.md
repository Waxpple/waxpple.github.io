---
title: "[Day16] RISC-V open hardware design"
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

# 先裝一些工具軟體

## SystemC on MacOS

### Version I (work)

到 Brew 直接無腦安裝 SystemC [Brew link](https://formulae.brew.sh/formula/systemc)

```bash
brew install systemc
```
安裝完長這樣：
![Imgur](https://i.imgur.com/yfkUvty.png)
然後就來測試一下 SystemC的 helloworld吧

```c++
// All systemc modules should include systemc.h header file
#include "systemc.h"
// Hello_world is module name
SC_MODULE (hello_world) {
  SC_CTOR (hello_world) {
    // Nothing in constructor 
  }
  void say_hello() {
    //Print "Hello World" to the console.
    cout << "Hello World.\n";
  }
};

// sc_main in top level function like in C++ main
int sc_main(int argc, char* argv[]) {
  hello_world hello("HELLO");
  // Print the hello world
  hello.say_hello();
  return(0);
}
```

```bash
g++ -o main hello.cpp -I/usr/local/include -L/usr/local/lib -lsystemc
```
執行：
![Imgur](https://i.imgur.com/ReIbiig.png)

### Version II (not working)
先到官方網站上下載systemC原始程式碼 [SystemC 下載](https://www.accellera.org/downloads/standards/systemc). 

> 參考網址 [Stackoverflow](https://stackoverflow.com/questions/25961573/how-to-use-and-install-systemc-in-terminal-mac-os-x)

![Imgur](https://i.imgur.com/CSoxj65.png)

然後解壓縮後執行以下步驟

```bash
$ mkdir build
$ cd build
$ export CXX=clang++
$ ../configure --with-arch-suffix=
$ make install
```
configure完應該長這樣：
![Imgur](https://i.imgur.com/toE1gRw.png)

結果直接不work xD 不知道為什麼...