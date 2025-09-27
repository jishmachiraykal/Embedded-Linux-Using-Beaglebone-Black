* initramfs is made of 3 words "'initial' 'RAM' based 'file system'"

* This is nothing but a file system hierarchy, made to live in the RAM of the device by compressing it (we use compression because RAM is precious, we cannot use whole RAM just to store the FS) and during booting, Linux mounts this file system as the initial file system.  That means you just need RAM to mount the FS and get going with the complete boot process

* Using initramfs is optional.Let’s say you have a product and your product has USB interfaces, mass storage devices like SD card and let’s say you also have networking peripherals like Ethernet and also display. Now, to operate this wide range of peripherals, all the device drivers must be in place right?

* That means all the drivers must be loaded in to the kernel space.And along with the drivers, some peripherals may require the firmware binaries to operate. One idea is make sure that all those drivers are “built in” in to the kernel, that’s not a great idea because that makes your Linux kernel specific to your product and it will drastically increase the Linux kernel image size.

* Another good way is, you come up with the minimal file system, where you store all your drivers and firmware, and load that FS in to the RAM and ask the Linux to mount that file system during boot( Thanks to kernel boot arguments , you can use the kernel boot arguments to indicate kernel that your FS resides in RAM)

* When the kernel mounts that file system from RAM, it loads all the required drivers for your product and all the peripherals of your product are ready to operate, because the drivers are in place. After That you can even get rid of this RAM based file system and use (switch to) some other advanced file system which resides on your other memory devices like eMMC/SD card or even you can mount from the network

* So, basically initramfs embedded into the kernel and loaded at an early stage of the boot process, where it gives all the minimal requirements to boot the Linux kernel successfully on the board just from RAM without worrying about other peripherals. And what you should store in initramfs is left to your product requirements, you may store all the important drivers and firmware, you may keep your product specific scripts, early graphic display logos, etc

* How to keep initramfs in to RAM? There are 2 ways,

    1) You can make initramfs “built in” in to the Linux Kernel during compilation, so when the Linux starts booting , it will place the initramfs in the RAM and mounts as the initial root file system and continues.

    2) You can load the initramfs from some other sources in to the RAM of your board and tell the Linux Kernel about it (i.e., at  what RAM address initramfs is present) via the kernel boot arguments

* How to generate initramfs? Download and extract the RFS attached in the lecture 47 of section 8 from TI software SDK.And place in uder fs in workspace. And ret in to the extracted folder and run the below 2 commands from the “terminal”
    1. $ find . | cpio -H newc -o > ../initramfs.cpio
    2. $ cat ../initramfs.cpio | gzip > ../initramfs.gz

* In the first command we are generating a cpio archive and then gz archive. Next step is to install mkimage command. For that run “apt-get install u-boot-tools” on your terminal software, this will install all the u-boot related tools along with mkImage tool

* Make our initramfs , uboot friendly by attaching the uboot header with load address and other info .Run the below command
```
    mkimage -A arm -O Linux -T ramdisk -C none -a 0x80800000 -n "Root Filesystem" -d ../initramfs.gz  ../initramfs
```

* Now you will end up with a file “initramfs” which also includes the uboot header and this file we will be used as ram based file system whenever required. Here embedded_linux/fs/initramfs