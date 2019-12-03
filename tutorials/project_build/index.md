---
title: Build a Project
layout: page
---
# Step by Step Build-Guide for the Zedboard
To get started using ReconOS, this guide leads you through the first steps to
setup your development environment. You will build the sort demo and execute
it on your board by following the step by step instructions given. The
SortDemo is an example application to demonstrate ReconOS and its concepts. It
uses both hardware and software threads to sort a bunch of data. The different
threads synchronizes via mboxes and access the data via the memory subsystem
of ReconOS. This guide includes the following steps:

* Prerequisites

* Boot the Linux Kernel
  * Compile U-Boot
  * Compile Linux Kernel
  * Setup Root Filesystem

* Run the SortDemo
  * Hardware Project
  * Software Project
  * Running the Demo


## Prerequisites

We assume that you have basic knowledge of the development for
an FPGA, especially for Systems on Chip, and that you have a working
installation of the appropriate tools and your development board:

* Linux workstation with a distribution of your choice (tested with Ubuntu 18.04), including
  * picocom
  * NFS server
  * Python 3.4 or greater
  
* Xilinx Vivado & SDK (Version 2019.1 for this guide)

* Evaluation board connected to your workstation
  (For this guide the Zedboard Rev. C or D)
  * JTAG connection to program the FPGA (optional)
  * UART connection to interact with the board

Furthermore, we need to download some external components as listed below.

* Linux Kernel: [https://github.com/xilinx/linux-xlnx](https://github.com/xilinx/linux-xlnx)

* U-Boot: [https://github.com/xilinx/u-boot-xlnx](https://github.com/xilinx/u-boot-xlnx)

* Busybox: [git://git.busybox.net/busybox](git://git.busybox.net/busybox)

For the Xilinx repositories we recommend to checkout the matching tag for your Vivado version. In our case this is `xilinx-v2019.1`, which will build the Linux kernel version 4.19.

### Setup Working Directory

At first you should clone all the repositories listed above and create a
folder for the root filesystem. `$WD` represents your working directory.

```
> cd $WD
> git clone https://github.com/reconos/reconos
> git clone https://github.com/xilinx/linux-xlnx
> git clone https://github.com/xilinx/u-boot-xlnx
> git clone git://git.busybox.net/busybox
> mkdir nfs
```

This should result in the following directory structure:

```
$WD
  \- reconos       -> the ReconOS repository ($RECONOS)
  \- linux-xlnx    -> the Linux kernel sources
  \- u-boot-xlnx   -> the U-Boot sources
  \- busybox       -> the busybox sources
  \- nfs           -> the root filesystem
```

## Boot the Linux Kernel

ReconOS builds up on an existing operating system. In this section you will
setup Linux and all required boot loaders to execute it on the Zedboard. Do
not worry if you have never cross-compiled your own Linux kernel before or
never booted a kernel using U-Boot. It sounds more horrific than it actually
is.

In the setup we will create in the following sections, the kernel itself is
located on the SD card and mounts the root filesystem via NFS during startup.
This allows to simply reboot the entire board by turning it off and on again
and gives you the flexibility to exchange files via network easily.

Before getting started, we want to briefly explain how the boot process on the
Zynq looks like. At first, the internal boot ROM is loaded. It setups the
system and executes the so-called First Stage Boot Loader (FSBL) dependent on
the jumper configuration. The FSBL initializes the hardware as configured by
the developer (`ps7_init`) and executes any provided program. In our case,
this is U-Boot in the role of a primary bootloader, which is responsible for
loading and booting the Linux kernel.

While the boot ROM is already stored at the development board, the other
executables involved in the boot process need to be provided by the developer.
Especially, the FSBL contains device specific code and is, therefore, provided
by Xilinx as proprietary software. However, the U-Boot project
developed an open source alternative called Secondary Program Loader (SPL).
Although not officially supported by Xilinx, we will use the SPL in this
tutorial, since it is integrated into the U-Boot build process and requires no
further configuration.

But now enough about the theory, let's get our hands dirty and be happy if we
see our first command prompt via UART.

### Build Environment

Cross-compiling U-Boot and the Linux kernel require some environment variables
to specify the target architecture and the appropriate cross compiler.
Therefore, export the following variables. You might need to adjust the cross compiler path depending on your SDK version.

```
export WD=<<path to your working directory>>
export ARCH=arm
export CROSS_COMPILE=/tools/Xilinx/SDK/2019.1/gnu/aarch32/lin/gcc-arm-linux-gnueabi/bin/arm-linux-gnueabihf-
export KDIR=$WD/linux-xlnx/
export PATH=$WD/u-boot-xlnx/tools/:$PATH
export PATH=$WD/linux-xlnx/scripts/dtc/:$PATH
```

### Compile Scripts

As preparation for compilation of U-Boot we need to build a tool (i.e. dtc) shipped with the linux kernel sources.
We will come back later to compile the kernel.
```
> cd $WD/linux-xlnx
> make xilinx_zynq_defconfig
> make scripts
```

### Compile U-Boot

As already said, the FSBL uses the `ps7_init` to configure the system. Since
we use the SPL as an FSBL replacement, we also need the `ps7_init` code here.
Fortunately, U-Boot already comes with a basic configuration which we will use
right now. However, note that if you change anything in your processing
system, you must recompile U-Boot and replace the `ps7_init` files with your
own ones.

The SPL is also capable of booting Linux directly but requires non-volatile
memory to store the kernel parameters. Since we do not want to setup this, we
will execute a full blown U-Boot instance, loading the kernel image and device
tree from SD-Card. Therefore, we need to disable the direct boot feature in
the configuration. In the latest U-Boot versions this option is now available
via the KConfig in `SPL / TPL -> Activate Falcon Mode`. You can choose the
preferred way of disabling this option, e.g. by editing the `.config` directly
or using `make menuconfig`.

Furthermore, we need to patch the boot command executed by U-Boot, since we do
not need to use a ramdisk. To do so, apply the following patch.

```
--- a/include/configs/zynq-common.h
+++ b/include/configs/zynq-common.h
@@ -284,8 +284,7 @@
                        "echo Copying Linux from SD to RAM... && " \
                        "load mmc 0 ${kernel_load_address} ${kernel_image} && " \
                        "load mmc 0 ${devicetree_load_address} ${devicetree_image} && " \
-                       "load mmc 0 ${ramdisk_load_address} ${ramdisk_image} && " \
-                       "bootm ${kernel_load_address} ${ramdisk_load_address} ${devicetree_load_address}; " \
+                       "bootm ${kernel_load_address} - ${devicetree_load_address}; " \
                "fi\0" \
        "usbboot=run xilinxcmd && if usb start; then " \
                        "run uenvboot; " \
```

We should also change the default boot order in this file.
```
--- a/include/configs/zynq-common.h
+++ b/include/configs/zynq-common.h
@@ -206,14 +206,14 @@
        "nor "

 #define BOOT_TARGET_DEVICES(func) \
+       func(XILINX, xilinx, na) \
        BOOT_TARGET_DEVICES_MMC(func) \
        BOOT_TARGET_DEVICES_QSPI(func) \
        BOOT_TARGET_DEVICES_NAND(func) \
        BOOT_TARGET_DEVICES_NOR(func) \
        BOOT_TARGET_DEVICES_USB(func) \
        BOOT_TARGET_DEVICES_PXE(func) \
-       BOOT_TARGET_DEVICES_DHCP(func) \
-       func(XILINX, xilinx, na)
+       BOOT_TARGET_DEVICES_DHCP(func)

 #include <config_distro_bootcmd.h>
 #endif /* CONFIG_SPL_BUILD */
```

Finally, to configure and build U-Boot, execute the following commands.

```
> cd $WD/u-boot-xlnx
> make zynq_zed_defconfig
> make menuconfig #disable Falcom Mode here
> make -j3
```

### Compile Linux Kernel

After we have compiled U-Boot, we can proceed with Linux. You will see, that
cross-compiling your own kernel is easier than you might think, since we will
just use the default configuration. If you wish, you can adjust the
configuration to your needs before compilation.

Additionally, Linux needs a device tree describing the underlying hardware and
including the kernel parameters passed during the boot. You need to adjust the
default device tree shipped with the kernel to fit our configuration.
The file for the Zedboard can be found in `$WD/linux-xlnx/arch/arm/boot/dts/zynq-zed.dts`
and the following diff shows the necessary changes.

The `bootargs` setting includes the kernel parameters passed during boot. We
specify the correct console and mount the root filesystem via NFS. Of course,
you need to adjust `<<hostip>>`, `<<path>>` and `<<boardip>>` to your setup.

Furthermore, you can see the ReconOS components added to the device tree.
These components include the correct addresses and used interrupt handlers.

```
--- a/arch/arm/boot/dts/zynq-zed.dts
+++ b/arch/arm/boot/dts/zynq-zed.dts
@@ -23,10 +23,31 @@
        };

        chosen {
-               bootargs = "";
+               bootargs = "console=ttyPS0,115200 root=/dev/nfs rw nfsroot=<<hostip>>:<<path>>,tcp,nfsvers=3 ip=<<boardip>>:::255.255.255.0:reconos:eth0:off earlyprintk";
                stdout-path = "serial0:115200n8";
        };

+       amba: amba {
+               reconos_osif: reconos_osif@75a00000 {
+                       compatible = "upb,reconos-osif-3.1";
+                       reg = <0x75a00000 0x10000>;
+               };
+
+               reconos_osif_intc: reconos_osif_intc@7b400000 {
+                       compatible = "upb,reconos-osif-intc-3.1";
+                       reg = <0x7b400000 0x10000>;
+                       interrupt-parent = <&intc>;
+                       interrupts = <0 58 4>;
+               };
+
+               reconos_proc_control: reconos_proc_control@6fe00000 {
+                       compatible = "upb,reconos-control-3.1";
+                       reg = <0x6fe00000 0x10000>;
+                       interrupt-parent = <&intc>;
+                       interrupts = <0 59 4>;
+               };
+       };
+
        usb_phy0: phy0@e0002000 {
                compatible = "ulpi-phy";
                #phy-cells = <0>;
```

Now you can compile Linux by the following make command. This might take a
while, so grab a coffee and cross your fingers.

```
> cd $WD/linux-xlnx
> make -j3 uImage LOADADDR=0x00008000
> make dtbs
```

### Build the root filesystem

To run Linux, we also need a root filesystem to mount. In this section we
will build a minimal root filesystem by compiling busybox. If you
do not want to build the root filesystem by your own, just download
it from here and extract it to $WD/nfs.

* Exemplary ARM root filesystem &#91;[rootfs_arm.tar.gz](rootfs_arm.tar.gz)&#93;

To create a minimal busybox setup, create a minimal config and enable all
features you like. After that, compile busybox and copy the generated files to
the root filesystem.

```
> cd $WD/busybox
> make defconfig
> sed -i "s|.*CONFIG_STATIC.*|CONFIG_STATIC=y|" ${WD}/busybox/.config
> make menuconfig # optional
> make -j3
> make install
> cp -r _install/* $WD/nfs
```

Besides busybox you must create some additional files and folders:

```
> mkdir dev etc etc/init.d lib lib/firmware mnt opt opt/reconos proc root sys tmp 

> cat > $WD/nfs/etc/inittab <<'EOF'
::sysinit:/etc/init.d/rcS

# Start an askfirst shell on the serial ports
ttyPS0::respawn:-/bin/sh

# What to do when restarting the init process
::restart:/sbin/init

# What to do before rebooting
::shutdown:/bin/umount -a -r
EOF

> cat > $WD/nfs/etc/init.d/rcS <<'EOF'
#!/bin/sh

echo "Starting rcS..."

echo "++ Mounting filesystem"
mount -t proc none /proc
mount -t sysfs none /sys

echo "rcS Complete"
EOF

> chmod +x $WD/nfs/etc/init.d/rcS

> ln -s bin/busybox init
```

### Setup NFS
As already mentioned, the root filesystem will be mounted via NFS. To allow
the development board to access the root filesystem, you have to create an
export for it by adding the following line to your `/etc/exports` file.
Replace `<<path>>`, `<<boardip>>`, `<<uid>>` and `<<gid>>` by the appropriate values.

```
<<path> <<boardip>>(rw,no_subtree_check,all_squash,anonuid=<<uid>>,anongid=<<gid>>)
```

Of course, you need to make sure to configure both the board and your
workstation properly to allow communication via network. This includes the
right ip addresses and a physical connection. Export the NFS share:

```
> exportfs -ar
```

### Compile ReconOS kernel module

ReconOS combines drivers in a kernel module which needs to be compiled and
copied together with a initialization script to the root filesystem.

```
> cd $WD/reconos/linux/driver
> make RECONOS_ARCH=zynq RECONOS_OS=linux RECONOS_MMU=true PREFIX=$WD/nfs/opt/reconos install
```

You can then simply initialize the entire ReconOS system by executing
`reconos_init.sh` on the ARM processor.

## Sort Demo

Until now, we have configured and installed a basic setup of our working
environment and now we are going to get in touch with the very first ReconOS
application, the well known SortDemo. You will see the toolflow of the ReconOS
Development Kit (RDK) and how to implement an entire application. To get
started with the RDK, the only thing you have to do is to source the settings
file under `$WD/reconos/setting.sh`. After that, you can simply type `rdk` to
start the development kit.

```
> source $WD/reconos/tools/settings.sh
```

To tell RDK what which Vivado version it should use and where it can be found, edit the `[General]` section of the configuration file at `$WD/reconos/demos/sort_demo/build.cfg`.

```
TargetXil = vivado,2019.1
XilinxPath = /tools/Xilinx
```

So let's take a look into the SortDemo project folder in
`$WD/reconos/demos/sort_demo`. It consists out of a source folder and a
project file describing the structure of the application. Out of these
sources, the RDK generates a complete Vivado project for the hardware design and
a ready to compile software project. To generate these two projects, simply
start the RDK and execute `export_hw` and `export_sw`. To get more information
for each command, you can execute it with the `--help` option or view the [documentation](/documentation/rdk).

```
> cd $WD/reconos/demos/sort_demo
> rdk export_sw
> rdk export_hw
```

Now, the RDK has created two new folders, `build.hw` and `build.sw`, which
contain the projects for hardware and software, respectively. To build both of
them, we again need to setup some environment variable and compile an
additional library. Again, the `CROSS_COMPILE` environment variable specifies
the compiler for the ARM processor used for the software compilation. The time
library is used by the SortDemo to get precise benchmarking results.

```
> export CROSS_COMPILE=/tools/Xilinx/SDK/2019.1/gnu/aarch32/lin/gcc-arm-linux-gnueabi/bin/arm-linux-gnueabihf-
> make -C $WD/reconos/linux/tools/timer
```

Now you can implement both projects.

```
rdk build_sw
rdk build_hw
```

The bitstream generation will take a while, so it might be the right time to
get a coffee.

### Running the Demo

Now you have everything you need to run the SortDemo on real hardware. Copy the software executable and implemented bitstream to the root filesystem.

Unfortunately, the FPGA manager included in newer Linux kernel versions does not support Vivado's .bin bitstream format directly, so you have to byte-flip the file using the xxd tool before moving it to the required directory.

```
> cp $WD/reconos/demos/sort_demo/build.sw/sortdemo $WD/nfs/opt/reconos
> xxd -e $WD/reconos/demos/sort_demo/build.hw/myReconOS.runs/impl_1/design_1_wrapper.bin | xxd -r > $WD/nfs/lib/firmware/bitstream_sortdemo.bin
```

Then setup the SD card shipped with the board. The only thing you have to
do, is to cleanup the card and copy the right files to it.

```
> cp $WD/u-boot-xlnx/spl/boot.bin /mnt/boot.bin
> cp $WD/u-boot-xlnx/u-boot.img /mnt/u-boot.img
> cp $WD/linux-xlnx/arch/arm/boot/uImage /mnt/uImage
> cp $WD/linux-xlnx/arch/arm/boot/dts/zynq-zed.dtb /mnt/devicetree.dtb
```

After that, insert the SD card into the Zedboard and configure the bootmode by
setting jumpers MI02, MI03 and MI06 to GND and MI04 and MI05 to 3V3. Turn on
the board, connect via UART and see how Linux boots. When the u-boot command prompt
appears, boot the Linux kernel with.

```
Zynq> boot
```

After Linux has booted, you can run the SortDemo.
```
/ # cd /opt/reconos
/opt/reconos # echo bitstream_sortdemo.bit > /sys/class/fpga_manager/fpga0/firmware
/opt/reconos # ./reconos_init.sh
/opt/reconos # ./sortdemo
/opt/reconos # ./sortdemo 2 1 32
```

### Optional: Mount filesystem from ramdisk image instead of NFS share

Although convenient during development, having the root filesystem mounted from NFS
is not always the best option. The Zedboard needs to be connected with a host PC at
all times in order to work. A different approach is to package the root filesystem
in a ramdisk image and mount that image from SD card. This section describes, how to
create and edit this image and how to setup U-Boot in order to start Linux with this approach.

First create an empty image file and mount it. You might want to increase the size (`count`) depending on your use case.

```
cd $WD/ramdisk
dd if=/dev/zero of=ramdisk.image bs=1024 count=16384
mke2fs -F ramdisk.image -L "ramdisk" -b 1024 -m 0
tune2fs ramdisk.image -i 0
chmod a+rwx ramdisk.image
mkdir $WD/ramdisk/mnt/
sudo mount -o loop ramdisk.image $WD/ramdisk/mnt/
```

Then copy busybox, init files, ReconOS kernel modules and create the image file in the correct
format.

```
sudo cp -r $WD/nfs/* $WD/ramdisk/mnt/
sudo umount $WD/ramdisk/mnt/
gzip $WD/ramdisk/ramdisk.image
mkimage -A arm -T ramdisk -C gzip -d $WD/ramdisk/ramdisk.image.gz $WD/ramdisk/uramdisk.image.gz
```

Revert the 1st patch for zynq-common.h and compile U-Boot again.

You also need to adjust the bootargs setting in the devicetree (zynq-zed.dts) and recompile it.
```
bootargs = "console=ttyPS0,115200 root=/dev/ram rw earlyprintk";
```

Copy the updated devicetree.dtb, u-boot.img, boot.bin and uramdisk.image.gz
to SD card. Now the Zedboard starts without relying on an Ethernet connection to a host PC. If you want you can mount the SD card itself and retrieve or store data this way:

```
/ # mount /dev/mmcblk0p1 /mnt
```
