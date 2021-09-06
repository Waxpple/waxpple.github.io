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

# Verilator

[參考來源 https://ys-hayashi.me/2020/12/verilator-2/](https://ys-hayashi.me/2020/12/verilator-2/)

## 安裝Verilator [Brew](https://formulae.brew.sh/formula/verilator).

用Brew安裝完，應該也沒什麼問題。

## 測試功能

先準備待測試的verilog code.

```verilog
module design_under_test(
    input clk, input rst,
    output logic valid,
    output logic [7:0] data
);
    logic [39:0] out_all;
    always_comb begin
        data = out_all[7:0];
        valid = (data != '0);
    end

    always_ff @(posedge clk or negedge rst) begin
        if (!rst) begin
            out_all <= "olleH";
        end else if (valid) begin
            out_all <= out_all >> 8;
        end
    end
endmodule
```

要跑的步驟如下

1. 撰寫 testbench `testbench.cpp`
2. 準備可合成之待測模組 `design_under_test.sv`
3. 把待測試模組轉換成 C++（或是 SystemC），並指名要使用的 testbench `verilator design_under_test.sv --exe testbench.cpp --cc`
4. Verilator 產生的檔案都在 `obj_dir` 資料夾（這個是預設名稱）。
5. 在該資料夾下面編譯產生 binary 
  `make -C obj_dir -f Vdesign_under_test.mk`。
6. 執行模擬 `./obj_dir/Vdesign_under_test`。

但是我們先跑一個空白的`testbench.cpp`看怎麼樣。

```bash
int main() {
    return 0;
}
```

存好`testbench.cpp`後，跑一次上面的流程，可以看到會generate一個資料夾 `obj_dir`下面有一個檔案`Vdesign_under_test.h`的前幾行大概是長這樣：

```c++
class Vdesign_under_test__Syms;

//----------

VL_MODULE(Vdesign_under_test) {
  public:
    
    // PORTS
    // The application code writes and reads these signals to
    // propagate new values into/out from the Verilated model.
    VL_IN8(clk,0,0);
    VL_IN8(rst,0,0);
    VL_OUT8(valid,0,0);
    VL_OUT8(data,7,0);
    ...
```

可以看出來 這四個I/O ports就是我們在verilog定義的。
因此我們testbench可以這樣寫：

```c++
#include "Vdesign_under_test.h"
#include <memory>
#include <iostream>
int main(int, char**)
{
    std::unique_ptr<Vdesign_under_test> dut(new Vdesign_under_test);
    dut->clk = 1; dut->rst = 1; dut->eval();
    dut->rst = 0; dut->eval();
    dut->rst = 1; dut->eval();
    for (int i = 0; i < 40; ++i) {
        dut->clk = 1 - dut->clk; dut->eval();
        if (dut->clk == 0 and dut->valid == 1)
        std::cout << char(dut->data) << std::endl;
    }
    return 0;
}
```
使用 `dut->eval()` 就會跳一次clock。

依照上面流程跑完應該會看到如下：
![Imgur](https://i.imgur.com/n2ECM1g.png)

# 本日小節

雖然我看這本書要寫的testbench是應該用 SystemVerilog去寫，然後我發現iverilog不支援 parallel assertion語法，就來研究一下verilator但是verilator要用SystemC來做testbench其實也不是我要的！所以要再survey看看。

附上今日迷因：
![meme](https://i.pinimg.com/736x/6a/56/eb/6a56eb98767dc0aab0136079493f4c96.jpg)