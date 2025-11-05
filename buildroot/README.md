### Buildroot introduction
* Buildroot is a simple, efficient and easy-to-use tool to generate embedded Linux systems through cross-compilation. https://buildroot.org/

* Documentation:https://buildroot.org/docs.html User manual: https://buildroot.org/downloads/manual/manual.html

* It is designed to run on Linux based systems

* Download buildroot from https://buildroot.org/downloads/ and place buildroot-2017.05.tar.gz under embedded_linux/downloads. Extract the tar file

* Get into buildroot dir
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

* Save and exit now

* Now build the buildroot
```
make -j4
make clean //if needed
```







