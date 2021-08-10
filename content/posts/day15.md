---
title: "[Day15] Linux on LiteX - iCESugar-Pro"
date: 2021-08-10T21:52:34+08:00
draft: false
---


# Install icesprog
icesprog is a tool which can program FPGA.
(https://github.com/FPGAwars/toolchain-icesprog)[https://github.com/FPGAwars/toolchain-icesprog]

```
sudo apt install git && git clone https://github.com/FPGAwars/toolchain-icesprog.git
cd toolchain-icesprog
./build.sh linux_i686
./build.sh linux_x86_64
./build.sh windows_x86
./build.sh windows_amd64
```

In my WSL settings, run `./build.sh windows_amd64` and restart command line tool.
Type `icesprog.exe` will show following message.

![Imgur](https://i.imgur.com/DTOgoQK.png)

# write bitstream file onto FPGA

```
git clone https://github.com/wuxx/icesugar-pro
icesprog.exe icesugar-pro/demo/linux-with-litex.bit
```

![Imgur](https://i.imgur.com/84vbYgL.gif)
This process will take some times, roughly 1 min.

After Done. We can disconnect the FPGA and prepare for the next step.

> you can use drag and drop the bitstream file onto FPGA iCELink and it will do the same magic!
# Prepare SD card for system image.
Take a microSD card, format it with FAT32 format. Assume SD card root is under `mnt/sdcard`
```
 cp icesugar-pro/linux/* mnt/sdcard/.
```

# Insert the microSD card on the back side of FPGA.

After finishing copy system file into SDcard. Insert it on the back side of FPGA.

![Imgur](https://i.imgur.com/C2vPTpS.jpg)

Connect the FPGA to PC and its led will glowing like this:
![Imgur](https://i.imgur.com/MRTdcfM.gif)

Check port number of the device.
![Imgur](https://i.imgur.com/Hs82ye2.png)

Open Putty on the PC. In serial line, type what you see above. For example, my FPGA is on COM4.

Set Speed(baudrate) to 10000000 and type session name `LITEX_LINUX` and click save.
Load `LITEX_LINUX` and click open.
> Next time, you can simply load the session without typing again.

![Imgur](https://i.imgur.com/a9VCPQy.png)

The Terminal will pop up like this:
![Imgur](https://i.imgur.com/3Ukks2u.png)

Press reset button on the FPGA and the terminal will load correct booting information.
![Imgur](https://i.imgur.com/UWs3bE4.gif)

That is all! Next step should be building our own system though liteX.