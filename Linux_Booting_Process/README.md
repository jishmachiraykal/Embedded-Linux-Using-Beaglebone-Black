* Linux boot requirements:
   RBL --> SPL/MLO --> U-boot --> Linux Kernel --> RFS

* RBL stands for ROM boot loader, is very first piece of code to run on the SOC when power is supplied to the board. This boot loader is written by the vendor(TI) and stored in the ROM of SOC during the production of the chip. This cannot be changed and we cannot get the source code of this. Major job of RBL is to load and run second stage bootloader such as SPL/MLO

* SPL(Secondary program loader)/MLO(Memory loader), the job of this bootloader is to load and execute third stage of bootloader i.e., U-boot from the DDR memory of the board. This runs out of internal SRAM

* U-boot: the job is to load and run the Linux kernel. Both U-boot and Linux kernel runs out of DDR

* To complete the successful boot we need RFS(Root file system)(SD/Flash/Network/RAM/e-MMC)

* SPL/MLO is actually derived from U-boot

* We can boot the AM335x SOC from the following boot sources

    1) NAND Flash

    2) NOR Flash (eXecute In place, XIP)

    3) USB

    4) eMMC

    5) SD card

    6) Ethernet

    6) UART

    7) SPI
 That means, you can keep the boot images in any of the above memory or peripheral and you can able to boot this SOC

* Ref table 26-7. SYSBOOT Configuration Pins in TRM, in that SYSBOOT[4:0].“SYSBOOT” is one of the register of this SOC and its first five bits decide the boot order.Ex:
```
   When SYSBOOT [4:0] = 00000b (This is reserved, you cannot use this configuration)

   When SYSBOOT [4:0] = 00001b

   After the RESET if SYSBOOT [4:0] = 00001b , then SOC will try to boot from UART0 first , if fails, then it tries to boot from XIP(XIP stands for eXutable In Place memory like NOR Flash), if that also fails, then it will try to boot from MMC0, if no success, finally it tries to boot from SPI0, if that also fails, then SOC outputs the error message and stops
```

* The ROM contains the ROM bootloader which runs when RESET is applied to the SOC. The code stored in the “ROM” is called ROM boot loader, this is programmed in to the ROM of the SOC during taping out of the chip, you cannot able to change it, why?? Because its ROM. Read only

* The job of the ROM is to set up the SOC clock, watch dog timer, etc and also its major job is to load the second stage boot loader, what we call MLO or SPL

* Now, from where it should load the second stage boot loader? For that what ROM code does is, it reads the register SYSBOOT [15:0], and based on the value of SYSBOOT[4:0] it prepares the list of booting devices. The register SYSBOOT [15:0] value is decided by the voltage level on the SYSBOOT pins

* That is, let’s say, if SYSBOOT[4:0] = 00011b, then boot order will be UART0,SPI0.XIP,MMC0 ( look at the table please !!! ) So, we can say that, The SYSBOOT pins configure the boot device order (set by SYSBOOT[4:0]).Some board, will give you the control to change the SYSBOOT[15:0] value by using dip switches

* SYS_BOOT2 is connected to a button S2 of the BBBB ( S2 is the boot button ). When you simply give power to the board, You will find the voltage level as below.

   SYS_BOOT0 = 0V

   SYS_BOOT1 = 0V

   SYS_BOOT2 =1V

   SYS_BOOT3 = 1V

   SYS_BOOT4 = 1V

* If you have Multimeter, measure the voltage of 45,44,43,41,40 pins of the expansion header P8 of the board you will find SYSBOOT[4:0] = 11100.when you press the button S2, SYS_BOOT2 will be grounded , so SYSBOOT[4:0]= 11000. Great! Now based on S2 (BBB boot button) we got 2 boot configurations

* 1) S2 Released (SYSBOOT[4:0] = 11100)

   The boot order will be

   MMC1 (eMMC)
   MMC0 (SD card)
   UART0
   USB0
   2) S2 pressed (SYSBOOT[4:0] = 11000) , The boot order will be

   SPI0
   MMC0 (SD card)
   USB0
   UART0
   So, to conclude, there are 5 boot sources supported for this board including SPI

* 1) eMMC Boot(MMC1) :eMMC is connected over MMC1 interface, This is the fastest boot mode possible, eMMC is right there on your board, so need not to purchase any external components or memory chip. This is the default boot mode. As soon as you reset the board, the board start booting from loading the images stored in the eMMC. If no proper boot image is found in the eMMC, then Processor will automatically try to boot from the next device on the list

* 2) SD Boot : If the default ( that is booting from eMMC) boot mode fails, then it will try to boot from the SD card you connected to the sd card connector at MMC0 interface. If you press S2 and then apply the power, then the board will try to boot from the SPI first, and if nothing is connected to SPI, it will try to boot from the MMC0 where our SD card is found. Also remember that we can use SD card boot to flash boot images on the eMMC. So if you want to write new images on the eMMC  then you can boot through sd card, then write new images to eMMC, then reset the board, so that your board can boot using new images stored in the eMMC

* 3) Serial boot : In this mode, the ROM code of the SOC will try to download the boot images from the serial port

* 4) USB BOOT :You may be familiar with this boot mode, that is booting through usb stick

* Boot sequence of RBL

* ROM code booting procedure can be found in 5029 of TRM: https://www.ti.com/lit/ug/spruh73q/spruh73q.pdf?ts=1757495289153&ref_url=https%253A%252F%252Fwww.ti.com%252Fproduct%252FAM3358

* Size of ROM in SOC is 176 KB, when board is powered on RBL will be the first component to run on the SOC. First it does the stack setup and calls the main(). It initializes the watchdog timer for approx. 3 minutes i.e., RBL couldn't able to load the SPL within 3 minutes watchdog timer will expire and reset the processor. Then setting the clock using PLL(Phase log loop), it is a clock generating engine using which we can generate wide range of clock frequencies for wide range of subsystems of the SOC. For PLL we need to give low frequescy oscillator like crystal oscillator, RC oscillator etc which in turn gives high frequency which can be used to run subsystem like display, processor, USB. 24 MHz(In-built value, look at the back side of the board Y2) ->  800 MHz

* Based on the value of 15:14 bits of SYSBOOT, RBL comes to know the value of external crystal oscillator connected to the SOC. SYSBOOT[15:14]=01b=24MHZ. Then PLL takes this i/p and produces much more high o/p which is of 500 MHz for A8. This frequency is fixed. Now RBL goes into booting, it fetches the second stage bootloader from device list as per the value from SYSBOOT[4:0]. Once it finds the SPL device to boot copies the MLO into internal RAM(SRAM) of SOC. SPL will have its own image header as decided by TI

* SPL and MLO are mostly same except the fact that MLO has a header which containts the info like load address, size of the image.. RBL reads the image header of MLO from that it will get 2 imp info like load address and total size of the MLO. Then copies it to intenal RAM of SOC. Finally it executes MLO/SPL

* Boot sequence of SPL

* SPL/MLO initializes SOC to a point that U-boot can be loaded into external RAM(DDR memory). Key work done by the MLO are:
   1. It does UART console intialization to print out the debug messages
   2. Reconfigure PLL to desired value
   3. Intializes DDR memory registers to use DDR memory
   4. Does muxing config of boot peripherals pin because its next job is to load the U-boot from boot peripherals
   5. If MLO is going to get the U-boot from MMC0/1 interface, then it will do the MUX configuration to bring out the MMC0 or MMC1 functionalities on the pins
   6. Copies U-boot image into DDR memory and passes control to it. U-boot image will also have own image header containing load address and size of the image
   7. MLO will not load the image into internal memory of SOC because its size is only 176 KB

* Can we avoid using MLO/SPL to load U-boot and directly load it into internal SRAM. Because size of SRAM is only 128 KB

* Can we use directly RBL to boot U-boot into DDR memory and skip using MLO or SPL. Answer is ROM code won’t be having any idea about what kind of DDR RAM being used in the product to initialize it.DDR RAM is purely product/ board specific.

   Let’s say there are 3 board/product manufacturing companies X,Y,Z

   X may design a product using AM335x SOC with DDR3. In which lets say DDR3 RAM is produced by microchip.

   Y may design its product using AM335x SOC with DDR2 produced by Transcend and Z may not use DDR memory at all for its product.

   So, RBL has no idea in which product this chip will be used and what kind of DDR will be used, and what are DDR tuning parameters like speed, bandwidth, clock augments, size, etc.

   Because to read ( or write) anything from/to DDR RAM , first, the tuning parameters must be set properly and DDR registers must be initialized properly .

   Every different manufacture will have different parameters for their RAM. So, that’s the reason RBL never care about initializing DDR controller of the chip and DDR RAM which is connected to chip.

   RBL just tries to fetch the SPL found in memory devices such as eMMC and SD card or peripherals like UART,EMAC,etc.

   And in the SPL/MLO, you should know, what kind of DDR is connected to your product and based on that you have to change the SPL code , rebuild it and generate the binary to use it as the second stage boot loader.

   For example the Beaglebone black uses DDR3 from Kingston and if your product uses DDR3 from transcend, then if the turning parameters are different then you have to change the DDR related header files and the tuning parameter macros of the SPL , rebuild and generate the binary

* We need to partition the micro SD card into 2, one is BOOT of FAT file system and other one is RFS of type EXT3/4

* Once the SD is connected to the board dmesg will show the message related to SD card connected and its product details. Open the gparted application in apps and look for SD device location at the top right corner. This is very important for partition. Ex: /dev/sda2

* Creating first file system: open gparted application right click on unallocated then select new give new size around 1000 MB label and click on add. To add the flags click on newly created partition and select flags add the flags such as boot in this case. Likewise add 2 partions for root and rootfs. After partition is created copy the MLO image downloaded from https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/pre-built-images/Angstrom_Demo/MLO-beaglebone-2013.04 to root partition. SD card will be in /media/hjz1kor/partition_name. In the partion MLO image name should be just MLO because bootloader will look only for that.

* Booting process of MLO from SD card is as follows:
   1. Put the board into power down mode by doing a long press on power button(S3). Press and hold it untill all LED's turns off
   2. Press and hold boot button(S2)
   3. Gently press and release the power button
   4. Release the boot button

* Open and check in minicom for the bootlogs of MLO. Here MLO will be calling u-boot.img i.e, next stage bootloader

* Next is to test the booting of u-boot image. To download u-boot image go to https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/pre-built-images/Angstrom_Demo/u-boot-beaglebone-2013.04-r0.img and place it in workspace downloads folder. Then copy it to root partition in the name of u-boot.img. Because SPL will look for this specific filename. Now press S2 button and give power. This will try to find the linux image

* The job of u-boot
   1. Initializes some of the peripherals like I2C, NAND, FLASH, ETHERNET, UART, USB, MMC because it supports loading kernel from all these peripherals
   2. Load the Linux kernel image from various sources to the DDR memory of the board.
   3. Boot sources: NAND, FLASH, ETHERNET, UART, USB, eMMC, serial port etc...
   4. Passing of boot arguments to the kernel

* We can pass uEnv.txt file which contains environment variables values fo the u-boot which directs u-boot how to behaviour. Any default behaviour can be overwritten using the file. U-boot looks for zimage(elf) + 64 bytes of u-boot image header. From image header u-boot gets lots of info about Linux kernel

* Pre-built linux kernel image can be found from https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/pre-built-images/Angstrom_Demo/Angstrom-systemd-image-eglibc-ipk-v2012.12-beagleboard.rootfs.tar.xz. After extracting under boot dir, there will be linux kernel images. All the extracted folders should be placed in rootfs partition

* Since u-boot.img will be in root partition, it will have no idea where the Linux kernel image is. To resolve this we need to use uEnv.txt to tell u-boot where Linux kernel image is. mmcboot=echo Booting from microSD ...; setenv autoload no ; load mmc 0:1 ${loadaddr} uImage ; using load command trying to load uImage from current directory to ${loadaddr}. ${loadaddr} is the address of Linux kernel

* SD card can be unmounted using umount /media/username/boot in the terminal. Then connect the board to PC and boot using SD card. Now you will get the login terminal. bootm ${loadaddr} - ${fdtaddr}. bootm is the place where u-boot hangs and passees the control to Linux kernel. In the bootlogs it displays the image information

* There is a C file which is responsible for decompressing of Linux kernel images

* Reading u-boot header information of uImage manually
    1. Load the uImage from memory device(SD card) into the DDR memory of the  board
    2. Use the memory dump command of u-boot to dump header information

* Go to u-boot prompt, run "load mmc 0:2 0x82000000 boot/uImage" --> mmc is the interface 0 is the SD card 2 is the second partition where uImage is present 0x82000000 is the DDR address. Once the command is run some bytes of memory will be loaded to DDR memory

* Next in u-boot prompt, md 0x82000000 4 // gives the value of first 4 header in little endian format i.e., magic number, checksum, image creation timestamp and size of the image etc... total 64 bytes of data. That's how we can check image information using u-boot command by RAM dumping. md is the command to do the RAM dumping. imi command gives the information about application image

* To load a fat based file system into memory use, fatload else use load

* Boot sequence of Linux kernel

* Linux kernel gets control from u-boot. u-boot handsover the control to head.s of the Linux boot starp loader. head.s calls misc.c(miscelleneous) which is also inside Linux boot strap loader used for umcompressing the compressed files.Then the control comes to the another head.s of Linux kernel. Then from head.d --> head_common.c --> main.c. From main.c control goes to first application of Linux i., INIT is launched

* How u-boot hands-off control to the boot strap loader of the Linux kernel? Let's explore the bootm.c file of u-boot source code.u-boot source code can be downloaded from https://ftp.denx.de/pub/u-boot/ (u-boot-2017.05-rc2). bootm.c can be found in rch/arm/lib/bootm.c location. This is the place where code to read the Linux kernel from memory verify the checksum, handing off control is implemented

* There is a boot_jump_linux(bootm_headers_t *images, int flag) function in the bootm.c file, in that there is a kernel entry funtion pointer. This is pointed to kernel_entry = (void (*)(void *fdt_addr, void *res0, void *res1, void *res2))images->ep; Here images->ep is the entry point address of Linux kernel which is assigned to kernel_entry. Next is r2 = (unsigned long)images->ft_addr; ft_addr is holding the RAM address at which "Device tree binary" is present in the DDR memory. This is assigned to a variable called r2.

* kernel_entry(0, machid, r2);; This code hands of control to the Linux. The entry point address is dereferenced. 0 is ignored by Linux. machid is the machine id of the board sent by u-boot to Linux. Passed to Linux via register r1. r2 is the ftd address stored in the DDR memory. This is where u-boot handing over control to Linux kernel

* This file(bootm.c) calls start routine in head.s of boot strap loader.

* Linux source code can be found in https://github.com/beagleboard/linux. head.s will be under arch/arm/boot/compressed/head.s. Search for start: routine from where the control comes to Linux kernel. mov r7, r1 saves the architecture ID/machine ID and mov r2,r8 saved the DTB address. This can be seen here kernel_entry(0, machid, r2) mchid is r1 and r2. Next go to arch/arm/boot/compressed/misc.c, search for decompress_kernel. In this function you can find Uncompressing kernel message same thing can be found in boot logs also

* Now control comes to Linux kernel from Linux boot starp loader and goes to another head.s located in arch/arm/kernel/compressed/head.s. This is a general arm specific code and does:
   1. CPU specific intialization
   2. Checks valid processor architecture
   3. Page table inits
   4. Initialize and prepare MMU for the identified processor architecture
   5. Enable MMU to supprt for virtual memory
   6. Calls start_kernel function of the main.c (Arch independent code)

* Both the head.s files are architectute(ARM) dependent

* head.s in Linux kernel will first check for proper processor type of the board. After knowing this, it will calls the processor specific initialization routines found in the respective processor specific files i.e., arch/arm/mm/proc**. All the processor specific calls made to initialize the MMU before turning it on. That means before giving the control to Linux generic code, MMU initialization and turining off/on MMU is must that is the duty of the architecture specific codes.

* start_kernel() is called from head_common.s located in the arch/arm/boot/compressed/head.s. From here flows come to the main.c of the Linux kernel

* Launch of INIT

* main.c is architecture independent generic file found in Init/main.c. Entry point for the main.c is start_kernel(). pr_notice("%s", linux_banner) prints the Linux version, compiler version, processor type, build number etc. linux_banner can be found in the version file under Init. start_kernel() does lots of operations such as mm initialization, schedule initialization, timer initialization, high resolution timer initialization before launching the very first application of the linux kernel

* At the end it calls a function called rest_init(). This lanuches the init function, it calls 2 threads kernel_thread(kernel_init, NULL, CLONE_FS), second one is kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES). Then it is starting the scheduler and then kernel is going to CPU idle loop. This is an infinite loop i.e., while(1){}. schedule_preemt_disabled(); cpu_startup_entry(CPUHP_ONELINE);

* kernel_thread(kernel_init, NULL, CLONE_FS) is a kernel thread used to execute/generate very first user application INIT. kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES) is also another kernel thread which is used to execute/generate other kernel threads

* Inside kernel_thread(kernel_init, NULL, CLONE_FS) function, the code free_initmem() reclaims the memory used by intialization function because those functions are no longer needed and no one going to call them. Can be seen in bootlogs with "Free init memory: 248K" clearing the RAM which can be used for other purposes

* Then the kernel_thread(kernel_init, NULL, CLONE_FS) function calls the init function try_to_run_init_process("/sbin/init") || if that fails try_to_run_init_process("/etc/init"), if this also fails then calls try_to_run_init_process("/bin/init"), last and next try_to_run_init_process("/bin/sh")

* Finally it lanches first user application init. If not found at least it will lauch shell application try_to_run_init_process("/bin/sh"). INIT is responsible for lauching other applications. PID number of the INIT program is 1. Busybox is not a bootloader. u-boot,grub,barebox are bootloaders.