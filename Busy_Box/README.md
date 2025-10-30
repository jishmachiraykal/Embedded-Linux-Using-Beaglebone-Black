### Busybox introduction

* Busy box(minimalist rootfile system) is software tool which enables to customise the root file system for our embedded products. If the product is resource limited in terms of memory then we can customize the RFS in such a way that it can fit into the product with limited memory. Busybox has the potential to significantly reduce the memory consumed by various linux commands by merging all linux commands in one single binary

* If we only consider bin and sbin directory then in Ubuntu we have 350+ commands. Total memory consumed by these directories is ~200MB. But busybox supports most of the commands which can be find in bin/ and sbin/ with minimal memory footprint

* Single executable memory which contains all the linux commands is busybox(ls,cat,ssh,ifconfig,mkdir,iptables,mv,rm,echo,wc,find,grep,insmod etc...). This logic is implemented as swtich case in C for each Linux commands

### Busybox compilation

* Go to https://busybox.net/about.html and click on Download Source. Downloaded https://busybox.net/downloads/busybox-1.26.0.tar.bz2 version and untar it. This is the source code of busybox(code for all the Linux commands)

* coreutils/ directory contains all the core utilities like cat, chmod, echo, sync, touch, uname, tee etc... modutils/ contains modules related codes for chmod, insmod, rmmod, modinfo, modeprobe(imp) etc... Find utilities like grep, find are present in findutils/

* Busybox compilation steps can also be followed from https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/notes/compilation_commands#L55

* Next step is to apply the default configuration
```
    make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- defconfig
```

* Change the default settings if needed from menuconfig
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

* Here we are building the Linux in a dynamic way instead of static way. For that open menu config, go to busybox setting by pressing "enter" button and select "build busy box as static binary(no shared libs)" option by toggling between spacebar to select and deselect and then exit the menuconfig. New configuration will be saved in .config files

* Next step is to generate busybox binary and minimal file system
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- CONFIG_PREFIX=<install_path> install
```

* In the install path, it will install the rootfile system of the built busybox. Install path can be anything. We can also give -j4 in the above cmd

* Busybox will generate 3 major directories bin, sbin and usr. These 3 are enough at least to boot the Linux.
cd bin/
```
ls -1 | wc -l
89

ls -1 /bin/ | wc -l
2060
```


* Default configuration generates arounf 89 commands, we can go to menuconfig to increase/decrease the commands. But the bin folder of Ubuntu PC contains around 2060 commands. Busybox has reduced it to 89 commands

* cd sbin/
```
ls -1 | wc -l
70

ls -1 /sbin/ | wc -l
533
```

* If you do ls -l in the bin folder of busybox, these commands are actually softlinks meaning they are actually pointing back to the busybox itself. There is only one command in the generated filesystem i.e., busybox(All the command are pointing to the busybox)

* Size of the file system will be(du -sh) will be very less. This size is static since we built using static method. If we do this using the dynamic way then it would be less than static file system size


### Kernel modules installation

* Now execute  from the Linux compilation steps by giving path of the rfs_static directory which was created before
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=<path of the RFS> modules_install
```

* Go to RFS folder, under lib directory we can see kernel version directory is created. Under 5.10.168/kernel/, we will find lots of subsystem. modules/5.10.168/modules.builtin is statically compiled kernel modules and modules/5.10.168/modules.dep is a very important file which list out all the dependencies between dynamically loadable kernel modules

* Below line in the modules/5.10.168/modules.dep file indicates that in order to install cdc_ether driver first we have to install usbcore driver and usb-common driver, otherwise installation will be unsuccessful. usbcore driver and usb-common driver will be depending upon other modules for installation if we check modules/5.10.168/modules.dep file
```
kernel/drivers/net/usb/cdc_ether.ko: kernel/drivers/net/usb/usbnet.ko kernel/drivers/usb/core/usbcore.ko kernel/drivers/usb/common/usb-common.ko
```

* modules/5.10.168/modules.dep is very important for modprobe command. modprobe command is used to add/remove modules from the Linux kernel. When we type modprobe cdc_eee.ko, first it will read modules/5.10.168/modules.dep file and understand the dependencies between these files. insmod is a imple program to insert a module into the Linux Kernel. But this will not read the dependecy file so it is better to use modprobe(insmod is not smart as modprobe)

### Testing boot images and busybox on bbb

* Create a directory called compiled_binaries under workspace(embedded) and copy MLO, u-boot.img and u-boot-spl.bin from u-boot-v2022.04-bbb.io-am335x-am57xx to it

* Also copy linux-5.10.168-ti-rt-r76/arch/arm/boot/uImage and linux-5.10.168-ti-rt-r76/arch/arm/boot/dts/am335x-boneblack.dtb to compiled_bins directory

* Keep the MLO and u-boot.img in the SD card. Load the uImage from the hostPC using TFTP protocol and mount RFS using NFS protocol. We should also keep uEnv.txt in the SD card to load uImage and dtb using TFTP protocol. Also mountRFS using NFS protocol

* Create a uEnv-nfs.txt file under compiled binaries folder and copy it to SD card as uEnv.txt(not uEnv-nfs.txt). Now in SD card, BOOT will have MLO, u-boot.img and uEnv.txt files

* Transfer uImage to /var/lib/tftp folder and create a folder under srv(under root dir) called nfs, and  under nfs create folder called bbb with sudo. In this location we have to copy all the RFS contents. Copy everything from rfs_static folder to bbb folder here

* Now open the file /etc/exports, this file grants access to various hosts or nfs clients to access the nfs based file system. In this file we have to add an entry "srv/nfs/bbb 192.168.7.2(rw,sync,no_root_squash,no_subtree_check)" Here 192.168.7.2 is the BBB IP address(address of nfs client). Save with sudo

* For nfs mounting, we have to run the below commands
```
sudo exportfs -a // run man exports to see what exports does
sudo exportfs -rv
sudo service nfs-kernel-server restart //starting the nfs service
```

* Execute sudo service nfs-kernel-server to check the status of nfs service. Remove SD card and connect SD card to BBB and boot from SD card. Now open minicom to see the logs


