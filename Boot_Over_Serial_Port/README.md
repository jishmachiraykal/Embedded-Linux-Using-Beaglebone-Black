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

* Now put the board into UART boot mode, for that press and hold S2(boot button) and then switch on power supply to the board. If the board is in UART mode then you see CCCC in the minicom emitted by RAM code of the SOC indicating that it is waiting to grab SPL via the UART. If you don't see SPL then target is not in UART mode

* To exit from the UART boot mode, either boot using SD card by pressing S2 and give power supply and wait until bootlogs and LED's shows up.Or boot from eMMC by don't pressing the S2 and just giving the power supply.It should boot from the eMMC. Now if you want to back to UART boot mode, then follow the above step

* Now file like spl, u-boot.img and kernel image needs to be transfered using xmodem. To go to xmodem, ctrl+a and s first to go the location where spl image is present in the host(https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/pre-built-images/serial-boot/u-boot-spl.bin) and select the file by pressing space bar once and hit enter. It will load the image and exit from xmodem. After that in the console, spl loaded logs will be shown and now it will be waiting for the u-boot.img. Using the same process from xmodem, load the u-boot.img(https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/pre-built-images/serial-boot/u-boot.img).Sometimes u-boot.img load will be incomplete, it might be that if you give more delay after spl load. If so, do it from the beginning(For me load of u-boot.img was successful, but after that there were no logs and was not able to enter into u-boot prompt as shown in the lecture,so the following steps are not done in the board)

* Once you enter u-boot prompt after loading u-boot.img, now its the time to transfer the linux image to the target using serial port.In the u-boot prompt, type loadx(stands for load over xmodem) recommended address at which we have to load the kernel image is 0x82000000. Therefore, "load 0x82000000" is the command in the u-boot prompt which loads any file using xmodem into the DDR RAM memory of the board

* After executing the loadx command, press ctrl+a and s to select the linux kernel image(https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/pre-built-images/serial-boot/uImage) from host PC using xmodem.Don'tremove the power from the board, if so everything will be vanished. Now execute "loadx 0x88000000" to send DTB file(https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/pre-built-images/serial-boot/am335x-boneblack.dtb) from host PC to target.Next step is to download the initramfs by executing "loadx 0x88080000" and load initramfs(https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/pre-built-images/serial-boot/initramfs) using xmodem

* Now we are ready to boot from the memory, but since we are using RAM based file system(initramfs), we have to tell that to the Linux kernel via boot arguments, otherwise Linux will not be having any idea where exactly the filesystem resided and boot fail(kernel panic). Now in order to tell this to Linux kernel execute the below set environment variable command. 0x88080000 is the location where initramfs is downloaded
```
setenv bootargs console=ttyO0,115200 root=/dev/ram0 rw initrd=0x88080000
bootm 0x82000000 0x88080000 88000000
```

* Now it should boot the kernel.Login screen will appear with name arm-355x. This can be changed to custom name and a logo appears which is provided by IT which can also be changed. This has to be changed in the some file of the RFS from where we have downloaded the image. Once after that create initramfs and repeat the procedute to get custom name and logo

* xmodem is the serial transfer protocol used by ROM boot loader to get SPL over UART and xmodem is the serial transfer protocol used by SPL to get uboot when you do serial boot. New version of u-boot supports, xmodem, ymodem, zmodem and other protocols to transfre the Linux kernel over UART0

* The ROM code will ping the host 10 times in 3sec, to start xmodem transfer. If the host doesn't respond UART boot will timeout.115200 is the baud rate ROM bootloader expects SPL with hw flow control disabled. Serial boot of the AM335x works over UART0 peripheral. We cannot boot the AM335x based boards via UART1 peripheral because ROM bootload doesn't support boot over UART1