---
title: "[Day17] RISC-V CPU IFU設計"
date: 2021-09-09T17:15:48+08:00
draft: false
---
# 前言
前面有一些心得，但還沒整理出來，這篇先寫關於IFU的設計理念。

# Verilog lint
本篇在攥寫的時候，都會嘗試使用spyglass 做rtl lint來確保rtl code quality. 假設 rtl code 如下

```verilog
module FA(
 input clk,
 input [3:0] a,
 input [3:0] b,
 output reg [4:0] sum
);

always@(posedge clk)begin
	sum = a+b;
end
endmodule
```

跑Spyglass lint之後會跟你說紅色地方要使用non-blocking.
![Imgur](https://i.imgur.com/D5lE7BV.png)

# 一些參考網址
https://blog.csdn.net/A_stranger/article/details/103532699

https://blog.holey.cc/2018/11/05/centos-7-eda-tools-installation/