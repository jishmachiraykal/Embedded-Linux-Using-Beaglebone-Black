There are 2 ways to create partition

* First one using fdisk
    1. sudo fdisk /dev/sdc
    2. m for help
    3. d to delete a partition, by default it is 2 and to delete one more partition just type d again
    4. Sometimes in order to reflect the deletion, reconnect the SD card to PC
    5. n to create a new partition,partition type should be primary and give min and max size in terms of bytes.For BOOT partition, size will be around 1GB and ROOTFS around 4GB
    6. Then select partition number(gide default value)
    7. To change the type of partition table(Linux and FAT), type t
    8. In that, C for FAT file type and 83 for Linux
    9. p to check the partions

* After that run sudo mkfs.vfat /dev/sdc* and sudo mkfs.ext4 /dev/sdc* for FAT file and Linux respectively

* Rename the partitions using sudo e2label /dev/sdX1 NewLabel for Linux ext partition and sudo fatlabel /dev/sdX1 NewLabel for FAT file. Status can be checked in sudo e2label /dev/sdX1

* Copy the contents to BOOT and ROOTFS directly

* If there is tar file we can use the above approach.The other approach is using disk image writer For .img downloaded from beaglebone website. .xz file should changes to .img using unxz image_name. Go to the .img file directory and open using "open with other applications" and select "disk image writer". In that, destination should be SD card and click on start restoring. You might get some warning related to partition size, need not to worry and proceed with restoring. Now it will format the SD card, and take the ROOTFS partition size on its own and copy the contents of .img to ROOTFS. This will have the ROOTFS partition and contents. Now to get BOOTFS partition, click on + icon at the bottom and give partition size(1 GB) and name(label/BOOT). This will create BOOT partition.Now copy the contents to BOOT ex: https://github.com/niekiran/EmbeddedLinuxBBB/tree/master/pre-built-images/SD-boot

* Now remove SD card and connect board to PC and boot via SD card. In the bootlogs you'll see something like /dev/mmxxx0p1 in the "boot" command which means it has booted using SD card(because SD card is MMC0). Can also be checked using lsblk command in the target