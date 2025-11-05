### Busybox introduction

* Busy box(minimalist rootfile system) is software tool which enables to customise the root file system for our embedded products. If the product is resource limited in terms of memory then we can customize the RFS in such a way that it can fit into the product with limited memory. Busybox has the potential to significantly reduce the memory consumed by various linux commands by merging all linux commands in one single binary

* If we only consider bin and sbin directory then in Ubuntu we have 350+ commands. Total memory consumed by these directories is ~200MB. But busybox supports most of the commands which can be find in bin/ and sbin/ with minimal memory footprint

* Single executable memory which contains all the linux commands is busybox(ls,cat,ssh,ifconfig,mkdir,iptables,mv,rm,echo,wc,find,grep,insmod etc...). This logic is implemented as swtich case in C for each Linux commands

* Busybox does not support ssh utility it has to be added by taking from third party sources. Hint : https://busybox.net/tinyutils.html

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

* If there is any no such file or directory error while booting from SD card after fetching Linux kernel using tftp. Create dev folder under rfs_static, then reboot again using sd card. Once the booting is done, busybox terminal will appear. Now if you do, ls under busybox terminal you will see bin  lib  linuxrc  sbin  usr(which was under rfs_static in host PC)


### Understanding busybox init and rcS script

* Linux finally launches init application as part of booting, if you do "ls -l init" under rfs_static/sbin, it will be pointing back to the busybox. There are 2 types of Linux init programs
```
  1. Busy box init: execute rcS script present in the /etc/init.d/ directory
  2. System V init: executes inittab script found in the /etc/init.d/ path
```

* rcS(S stands for start) is mainly used to execute any services needed to start the Linux kernel application. Ex: Start networking, start sshd, start NFS, start deamons, start daemons, load modules, mount FS etc..

### Integrating rcS(Startup) scripts

* Create etc/init.d directory under rfs_static. Create rcS file under this(See lec 75 for content under section 13). When rcS file is launched it executes all the script starting with S(S01logging, S40network, S50sshd). Here number stands for sequence(it executes 01,40,50 in s sequence)

* Create rcK file under rfs_static/etc/init.d to shutdown the services.Keep S01logging, S40network, S50sshd scripts also in the path where rcS and rcK are present. Refer sec 13 lecture 75 for its contents

* Also create proc folder under rfs_static for mounting. Now boot the board, first it will mount proc and also error related to rcS file will be gone now. "ifup -a" command will look for /etc/network/interfaces file

### Enabling Ethernet over USB by driver integration

* Go to downloads/linux-5.10.168-ti-rt-r76 and exectute below command
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- INSTALL_MOD_PATH=rfs_static modules_install
```

* With this you will get the kernel modules under lib in rfs_static. Now to load kernel modules to target and enable Ethernet over USB, in the busy box terminal run the below comamnd
```
modprobe g_ether
```

* Now, if lsmod is executed we can see different drivers installed for USB. Now do "ifconfig usb0 192.168.6.2 up", after this ifconfig will show the usb0 interface

* While doing usb interface up/down, you will see the wifi getting connected/disconnected. Due to issue with ethernet, this(tftp) protocol is not working, hence busybox not tested exactly


### Autoloading of drivers during system startup

* This is to automate the process of bringing USB0 inetrface on hw. Open busybox terminal, mkdir etc/network/interfaces, then in the interfaces file add the commands like below to bring the interfaces up. Ex:
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
   address 192.168.7.2
   netmask 255.255.255.0
   network 192.168.7.0
   gateway 192.168.7.1

auto usb0
iface eth0 inet static
   address 192.168.6.2
   netmask 255.255.255.0
   network 192.168.6.0
   gateway 192.168.6.1
```

* Before bringing the interfaces up, we need to load the drivers. Write a script(Refer section 13 lecture 77), for example S02module
```
# need to add some more commands, basics would be
    start)
    sbin/modprobe g_ether

    stop)
    rmmod g_ether
```

* Keep the above file under rfs_static/etc/init.d in the host.

* In the busybox terminal, "touch /etc/network/if-pre-up.d"

* Create "mkdir var", "mkdir run" and "touch ifstate.new"

* Now reboot the board and usb0 should be up

* Busybox is the final image name, generated by busybox compilation which comprises all the selected commands

* rc6 run level directory scripts in /etc/ will be executed when you reboot your Linux PC
