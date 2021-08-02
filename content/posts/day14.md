---
title: "[Day14] COSCUP waxapple wandering"
date: 2021-08-01T11:23:43+08:00
draft: false
---
# Skywater 130nm PDK OSS

![Imgur](https://i.imgur.com/bYGhJiu.png)
# eFabless
![Imgur](https://i.imgur.com/UMSvkOF.png)
![Imgur](https://i.imgur.com/gbKTYKZ.png)
![Imgur](https://i.imgur.com/b83uNFO.png)


# OpenLane

One push button to start your ASIC design!

![Imgur](https://i.imgur.com/YcgY7pQ.png)

# Share note

### The efabless Caravel project---Chip design for the software-oriented - Tim Edwards, Mohamed Shalan

###### tags: `COSCUP2021` `Skilled` `en` `COSCUP2021` `Bringing Open Source Software to Hardware` `TR313`

{%hackmd kra72OaxRTiBzdV8Y4GMKA %}

:::info
Slido: https://app.sli.do/event/uq1d7voh
:::

Hardware 的困難：
1. require EE degree
2. cycle time
3. cost

Open Source EDA -> 需要封閉的 PDK

Skywater water-pdk => SKY130 data

# 成果

caravel picoRV32 搭配 closed source chips
EDA tool: OpenLane

10 mm^2 的 user project space
可以插入 user chip 跟 picoRV32 搭配

# OpenLane

RTL to GDSII open source tool

## Synthesis
verilog/VHDL + Standard cell library (SCL) => gate

## Floor and Power Planning
大致決定各部件的擺放位置
Power metal structure

## Place
Place gates
* Global place -> 大致位置
* Detail place -> aligned one grid

## Clock Tree Synthesis (CTS)
H tree or X tree 確保 clock 信號在 chip 都同步

## Routing
佈線，將 gate 用 Metal 線連起來

## Signoff
DRC + LVS

```
+      +

 \____/
 
```

OpenLane 希望達到一鍵完成上面所有步驟

Caravel all done in open source tool:




# SOFA Skywater opensource FPGAs

[https://github.com/lnis-uofu/Sofa](https://github.com/lnis-uofu/Sofa)



## Slack group for skywater PDK

裡面是參加eFabless專案的Slack，可以在裡面問任何想問的問題。
[https://invite.skywater.tools/](https://invite.skywater.tools/)


# Some useful links

[1] https://github.com/chipsalliance/verible

[2] https://github.com/idea-fasoc/OpenFASOC

[3] https://efabless.com/design_catalog/asic_platform/116

[4] https://github.com/efabless/caravel_user_project

[5] https://chipsalliance.org/workgroups/

[6] https://docs.google.com/presentation/d/e/2PACX-1vQwsMbjsRRKeelhQ5Rjz2jnV1Y90V7g-194o_1UMWSHFbzw1Kuq5DVrK4WwI_5GPA4kgAdBF3OVBoWV/pub?start=false&loop=false&delayms=3000&slide=id.g459757883e_0_2391