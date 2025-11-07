### Buildroot introduction
* Buildroot is a simple, efficient and easy-to-use tool to generate embedded Linux systems through cross-compilation. https://buildroot.org/

* Documentation:https://buildroot.org/docs.html User manual: https://buildroot.org/downloads/manual/manual.html

* It is designed to run on Linux based systems

* Download buildroot from https://buildroot.org/downloads/ and place buildroot-2017.05.tar.gz under embedded_linux/downloads. Extract the tar file

* Get into buildroot directory
```
arch: contains configuration of different architecture
configs: configuration related to different boards in the market(beaglebone_defconfig is need for configuring and building)
docs: consists of manual, website etc
board: directories related to lots of boards(ls board/beaglebone/patches will be applied to the Linux kernel whenever required by the buildroot itself). ls board/beaglebone/readme.txt shows how to use beaglebone black/how to build etc...
```

### Configuring and building buildroot

* Referred from embedded_linux/downloads/buildroot-2017.05/board/beaglebone/readme.txt. Steps to build are
```
make beaglebone_defconfig # .config will be generated

make menuconfig
```

* The above command will open menuconfig gui and go to toolchain and under toolchain type select external toolchain. The external toolchain backend allows to use existing pre-built cross-compilation toolchains. Buildroot knows about a number of well-known cross-compilation toolchains from Linaro for ARM, Sourcery CodeBench for ARM, x86-64, PowerPC, and MIPS, and is capable of downloading them automatically, or it can be pointed to a custom toolchain, either available for download or installed locally

* Otherwise if buildroot toolchain option is set, then it will generate its own toolchain

* In the toolchain origin, select "Pre-installed toolchain" and save

* In the toolchain path, give the path of gcc toolchain which is already installed from /home/user/embedded_linux/downloads/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf

* Under system configuration, change the system hostname and banner if you want and in the init system, select busybox(there are two types of init busy box and system) and in the "Run a getty after boot" option select the baudrate to 115200, tty port to ttyO0

* Under Kernel, deselect Linux Kernel option as we have already built the Kernel and under target packages, in network applications select "openssh" support

* Under bootloaders, deselect U boot as it is done already

* Save and exit now. There was an error related to Kernel version 4.10.xx used by /home/user/embedded_linux/downloads/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf. Then https://releases.linaro.org/components/toolchain/binaries/6.3-2017.02/arm-linux-gnueabihf/gcc-linaro-6.3.1-2017.02-i686_arm-linux-gnueabihf.tar.xz version was downloaded and given as toolchain path in menuconfig

* Now build the buildroot
```
make -j4
make clean //if needed
```

* Build was failed due to GNU lib error.

* Once the build gets completed successfully, go to output directory under buildroot folder from where build was done, rootfs.tar file will be generated. Extract this file into rfs_static under downloads and take backup of rfs_static before removing the previous contents. Now boot BBB and it should boot from nfs and should be able to see the banner that was set in menuconfig after the boot

* To enable the ssh access, go to etc/ssh folder under rfs_static and in sshd_config, uncomment permitlogin and remove prohibit password and give yes. Now restard sshd service on bbb, to do that go to etc/init.d and run "./S06sshd restart"

* Now in the hostPC do, ssh -l root 192.168.7.2

* To get access to hostPC from BBB, install openssh server(apt get install openssh) and in BBB "ssh -l root 192.168.7.1". Like this we should be able to access hostPC from BBB

* This is the power of buildroot, we can add/remove as many application we want

### Buildroot Linux and U-boot configurations

* Building Linux and U-boot from buildroot, go to top level directory of buildroot and "make menuconfig"

* In the kernel, select Linux kernel and in the Kernel version by default it selects latest version it might not work sometime and stable version of Linux for BBB can get from https://github.com/beagleboard/linux/ and give this url in the custom repo section

* For now going with latest version(4.11.3) and in the defconfig name give omap2plus(got from Linux compilation steps). Kernel binary format image will be U-image. Load address is 0x80008000. Select "Build a device tree blob" option

* Device tree source file name should be "am335x-boneblack". Done with Kernel, now direct to bootloaders

* Select U boot, in u-boot board name give give am335x-boneblack. Select U boot needs dtc option and U boot binary format should be u-boot.img. Select "Install u-boot SPL image name" and name should be "MLO". Save and exit. Execute

```
make -j4
```

* Got same error what was seen in build without Linux kernel and U-boot. If the build is successful but if still there is "no such file or directrory for MLO/zImage" ignore because this was not selected

* After the successful build, from buildroot top level directory go to output/images and we can see u-boot, rootfs.ext, MLO and am335x-boneblack.dtb images

* Transfer U-Image and am335x-boneblack.dtb to /var/lib/tftpboot and copy of rootfs is not needed here because we have not changed anything

* Reboot the board and now it should take 4.11 Kernel. Can be checked using uname -r inside the target





