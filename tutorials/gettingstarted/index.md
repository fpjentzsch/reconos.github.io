---
title: Getting Started
layout: page
---
# Quickstart with Prebuilt Demos

To get a first impression of ReconOS without fiddling around with
the development and build process, we provide prebuilt images for
the ZedBoard development board, including two example applications.
You can simply download and exctract them to your SD card and have
a working ReconOS system. Detailed instructions how to set up your
board are provided in the following sections.

## Demo Descriptions

### Sort Demo
The sort demo is the well known demo application for ReconOS. It sorts a
bunch of data by using both hardware and software threads and demonstrates
the possible performance benefits from using hardware threads.

The sort demo takes three parameters, the number of hardware threads (1 or 2),
the number of software threads and the number of blocks you want to sort.
As a result, you get a report of the sorting times.

### Matrix Multiplication
The matrix multiplication demo multiplies two matrices with a variable number
of software and hardware threads using the Strassen algorithm.

The demo takes three parameters, the number of hardware threads (1 to 4), the
number of software threads and the size of the matrices to multiply (256, 512
, 1024 or 2048).

## ZedBoard
### Required Resources
* Boot files &#91;[zedboard.tar.gz](zedboard.tar.gz)&#93;

### SD Card and Board Setup
After downloading the boot files, extract them to the
SD card shipped with your ZedBoard. Then insert the SD card into
the ZedBoard slot on the right and adjust Jumpers `MIO2` to `MIO6`
to the follwing configuration:

| Jumper   | Function                              | Setting |
|----------|---------------------------------------|---------|
| `MIO2`   | JTAG Mode (Cascaded, Independent)     | `GND`   |
| `MIO3`   | Boot Device (JTAG, Quad-SPI, SD Card) | `GND`   | 
| `MIO4`   | Boot Device (JTAG, Quad-SPI, SD Card) | `3V3`   | 
| `MIO5`   | Boot Device (JTAG, Quad-SPI, SD Card) | `3V3`   | 
| `MIO6`   | PLL Mode                              | `GND`   |

If not already done, connect the UART to your PC, turn the board
on and connect via a terminal emulator to your board using a baud rate
of 115200, for example using picocom:

```
> picocom -b 115200 /dev/ttyACM0 
```

U-Boot should initiate the boot process automatically. If interrupted, you can start it manually:

```
Zynq > boot
```

A couple seconds later the Linux kernel will great you with a new prompt.
Change directories and start playing with the demos. For
that, first program the FPGA with one of the demo bitstreams, then run the ReconOS
init script and finally the demo application.

```
/ # cd /opt/reconos
/opt/reconos # echo bitstream_sortdemo.bin > /sys/class/fpga_manager/fpga0/firmware
/opt/reconos # ./reconos_init.sh
/opt/reconos # ./sortdemo 2 2 32
```

Or for the matrix multiplication demo run.

```
/opt/reconos # echo bitstream_matrixmul.bin > /sys/class/fpga_manager/fpga0/firmware
/opt/reconos # ./reconos_init.sh
/opt/reconos # ./matrixmul 4 2 256
```
