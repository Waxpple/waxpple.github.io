---
title: "[Day11] Icesugar-Pro Development!"
date: 2021-07-24T23:20:22+08:00
draft: false
---

# Install Project Trellis
The FPGA use EPC5 chips from lattice. We can use Trellis+ yosys+ nextpnr to develope our design.
[https://github.com/SymbiFlow/prjtrellis](https://github.com/SymbiFlow/prjtrellis) 

This section will takes more than an hour to make it all done.

## Install the dependencies for Project Trellis

### Install Boost

```bash
sudo apt-get install libboost-all-dev
sudo apt install build-essential libboost-system-dev libboost-thread-dev libboost-program-options-dev libboost-test-dev
sudo apt install  libboost-filesystem1.71-dev
```
### Install cmake

```bash
sudo apt install cmake
```

### Install gcc-9

```bash
sudo apt install gcc-9
```

### Install Eigen3

```bash
sudo apt-get install libeigen3-dev
```

## Install Project trellis

```bash
 git clone --recursive https://github.com/YosysHQ/prjtrellis
cd prjtrellis/libtrellis
cmake -DCMAKE_INSTALL_PREFIX=/usr .
make
sudo make install
```

## Install nextpnr

```bash
git clone https://github.com/YosysHQ/nextpnr
cd nextpnr
cmake . -DARCH=ecp5 -DTRELLIS_INSTALL_PREFIX=/usr .
make -j$(nproc)
sudo make install
```

This step takes long time to make, especially large ram usage. If failed, you can see [https://github.com/YosysHQ/nextpnr/issues/115](https://github.com/YosysHQ/nextpnr/issues/115) for more detailed build information.