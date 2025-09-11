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