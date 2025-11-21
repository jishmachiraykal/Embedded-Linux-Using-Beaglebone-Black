* Take the latest debian image from https://www.beagleboard.org/distros

* Taking https://www.beagleboard.org/distros/am335x-11-7-2023-09-02-4gb-microsd-iot image for now

* Download the pre-built images for SD-Card from https://github.com/niekiran/EmbeddedLinuxBBB/tree/master/pre-built-images/SD-boot and copy it to the BOOT partition. After that run sync to make sure all the buffer content is flashed to the /media

* Then go the the debian image downloaded path do unxz image_name.A .img file will be created and once doing that go to file explorer, click on .img file and select open with disk image mounter. If you don't find 'disk image mounter' option, select open with other application and there we select 'disk image mounter'. Then go to terminal and execute lsblk command. Find the path to /rootfs. Go to that path and copy all the contents to ROOTFS partition of SD card

* Remove the SD card from PC and connect board and boot using SD card. There will be bootlogs from SPL-u-boot-Linux-kernel etc... Login using credentials and execute lsb_release -da, it prints the distribution specific information. uname -r gives the kernel specific version

* Checking current OS/kernel version in BBB. Use one of the following in terminal. They give different level of details.
```
    cat /etc/os-release
    lsb_release -a
    hostnamectl
    uname -r to get just kernel
```