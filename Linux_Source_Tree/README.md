* Linux source code can be found in https://github.com/beagleboard/linux/tree/v5.10.168-ti-rt-r76 and kept under downloads folder in workspace

* Core arm related codes like memory management, cache memories, exceptions etc... will go into arch/arm/kernel, arch/arm/mm, arch/arm/compressed, arch/arm/boot/lib

* Code specific to SOC will go into which has prefix mach-*(mach-omap2 TI's code) in arch/arm. Every vendor of the SOC has their own directory. This machine specific directory has two parts i.e., board_file and machine shared source code

* Board file is a C source file which explains various peripherals on the board(outside SOC). This file usually contains registration function and data of various on board devices ex: ethernet phy, eeprom, leds, lcds etc...

* Machine shared source code contains various C source files which contain helper function to initialize and deal with the various on chip peripherals of the SOC and these codes are common among all the SOCs which share common IP for the peripherals. These files can be changed accoring to the product design and these files are common among all the SOCs which share the same IP as omap2 SOC

* After the introduction of DTB(device tree binary), there is no separate board name for board file. Board files will be in order arch/arm/mach-omap2/board-generic.c This single file contains all the information related to the board through which board value gets initialized

* Now there is only one init i.e., omap_generic_init(). So the DTB will have multiple board related initialization ex: using macros we can enable/disable it. So we need to change the DTB according to the board/platform

* When the Linux detects DTB it reads the field called .dt_compat(.dt_compat = omap242x_boards_compat) then it will run a comparision function where it will compare this value with the every machine registration(ti,omap2420) and if there is a match found it gets the platform and will do the initialization

* drivers code can be found under drivers/ directory in embedded_linux/downloads/linux-5.10.168-ti-rt-r76 and is one of the biggest layer of the Linux kernel. It contains driver codes for various memory devices, peripherals, interfaces, bus controllers, networking devices, usb, graphics, lcd, crypto engine etc... Drivers will be given by the vendor(TI here). Vendors have the own maintainers who maintain the respective drivers for them

* driver for the LCD controller can be found under drivers/video/fbdev/ and for touchscreen under drivers/input/touchscreen/ These provide the interface between SOC and peripherals like SD/MMC/SDIO cards etc...

### Configuring and generating Linux image

* Steps can also be found in https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/notes/compilation_commands#L34

* Step 1: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean

* Step 2: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- omap2plus_defconfig (4.11)
for 4.4 use omap2plus_defconfig. .config will be created and should not edit the file. To edit it run menuconfig shown below

* Step 3: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig. In the menuconfig it you see any option with * means linux image will consist of this module and option M will not be part of final generated Linux image. So this has to be compiled separately and dynamically load the module to Linux kernel. We can also use M(loadable module) to minimize the size of Linux image. If something is changed in menuconfig, it will be reflected in .config file.

* CDC EEM Support under menuconfig is responsible for ethernet over USB of the board

* Step 4: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- uImage(output name of the kernle image) dtbs(DTB) LOADADDR=0x80008000(load address in the RAM, used by the u-boot) -j4. First it will run zimage and then it runs mkImage command to attach the uImage. Output is given below
```
    OBJCOPY arch/arm/boot/zImage
    Kernel: arch/arm/boot/zImage is ready
    UIMAGE  arch/arm/boot/uImage
    Image Name:   Linux-6.14.0-13039-ge8b471285262
    Created:      Wed Oct  1 10:53:37 2025
    Image Type:   ARM Linux Kernel Image (uncompressed)
    Data Size:    5435000 Bytes = 5307.62 KiB = 5.18 MiB
    Load Address: 80008000
    Entry Point:  80008000
    Kernel: arch/arm/boot/uImage is ready
```

* Step 5: ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 modules //generate loadable kernel modules and generates .ko files

* Step 6: make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=<path of the RFS> modules_install(path of the RFS obtained from busybox build. Don't execute now if its not generated). Installs all the modules which has to be transfered to RFS

* Cross toolchain is used on the host to generate binary for target processor architecture.Every SOC doesn't require SPL/MLO, its TI specific. mkImage tool is used to u-boot header for a image

* Note: If there is any "no such file or directory" error for gmp.h and mpc.h try to install development packages like sudo apt-get install libgmp-dev and sudo apt-get install libmpc-dev respectively