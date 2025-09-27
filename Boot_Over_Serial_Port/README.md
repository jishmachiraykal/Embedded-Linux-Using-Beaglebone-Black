* Serial booting means transfering the boot images from host to target using serial port(UART) in order to boot the target.

* Now boot from UART by pressing S2 button, because boot sequence taken when S2 is pressed is SPI0, MMC0, USB0, UART0. So remove SD card, USB cable so that it goes to UART and boots from there

* USB cable should be removed because otherwise we need to wait for 4.5 minutes to boot from UART. So in order to avoid connect power adapter and boot from serial port

* UART boot procedure can be found in the TRM under 26.1.9.5 https://www.ti.com/lit/ug/spruh73q/spruh73q.pdf?ts=1757495289153&ref_url=https%253A%252F%252Fwww.ti.com%252Fproduct%252FAM3358

* When we keep the board into UART boot mode,the ROM bootloader is waiting for the second stage i.e., SPL image over xmodem protocol only

* Once the SPL executes, it tries to get the third stage bootloader i.e., uboot image over xmodem protocol and uboot image should be sent over xmodem protocol from host

* When uboot executes uboot commands such as xmodem/ymodem to load rest of the images like linux kernel images, DTB, initramfs into the DDR memory of the board at the recommended address. Recommended load addresses are:
    Linux kernel image   - 0x82000000
    FDT or DTB           - 0x88000000
    RAMDISK or INITRAMFS - 0x88080000

* If you are facing issues with uboot boot after downloading it through XMODEM, then please refer to these threads where TI Software team suggests to use YMODEM protocol to download the uboot image instead of XMODEM. https://e2e.ti.com/support/arm/sitara_arm/f/791/t/646278?AM3358-UART-boot-mode and http://processors.wiki.ti.com/index.php/AM335x_U-Boot_User%27s_Guide#Boot_Over_UART

* Serial boot images can be found in https://github.com/niekiran/EmbeddedLinuxBBB/tree/master/pre-built-images/serial-boot

* Now put the board into UART boot mode, for that press and hold S2(boot button) and then press and release S3(power button). If the board is in UART mode then you see CCCC in the minicom emitted by RAM code of the SOC indicating that it is waiting to grab SPL via the UART. If you don't see SPL then target is not in UART mode.