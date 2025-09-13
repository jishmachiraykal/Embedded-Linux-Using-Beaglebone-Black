* BBB comes with in-build 4GB eMMC memory. eMMC is connected to mmc1 interface. By default BBB will boot from mmc1 interface when powered up

* We need to re-flash the eMMC memory to boot from eMMC by the help of SD card. Process is as follows:
    1. Download the latest debian OS
    2. Write the bootable image to the SD card
    3. Boot the board from SD card
    4. Execute emmc flasher script present in the SD card
    5. Flasher script will flash all the contents from SD card memory to eMMMC memory

* Flashing takes 5-10 minutes. Once it is done SD card can be removed from board and BBB will boot from eMMC. This has be done whenever beaglebone.org releases new version

* Flasher script will devide the eMMC memory into two partitions called BOOT and ROOTFS. Then it will format the partitions to create file system(FAT & EXT4). Then copy the contents of SD card into newly created partions

* Check the debian OS image using lsb-release -da and latest version will be in https://www.beagleboard.org/distros