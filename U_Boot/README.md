### Understanding of U boot source tree
* Download/Clone the u-boot source code from https://openbeagle.org/beagleboard/u-boot/-/tree/v2022.04-bbb.io-am335x-am57xx and place it into downloads folder of workspace

* This u-boot source supports various architecture and various boards. Architecture of various codes are stored in arch folder. board folder has list of supported boards and ti folder contains different folders for different SOCs. BBB runs with am355x SOC. All the board related file can be found here. board.c inside am355x directory takes care of intializing the values for BBB

* CPU specific initialization can be found under arch/arm/cpu. ls u-boot-2017.05-rc2/arch/arm/cpu/armv7/start.S this is the point where ROM bootloader handsover the control to SPL(second stage bootloader). start.S goes from CPU initialization to SOC initialization. configs directory contains configuration of the different boards.Every file ends with defconfig meaning default configuration. In our case we are using am335x_evm_defconfig as config file

### Cross toolchain installation
* Download the latest toolchain from https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/ and the one which is downloaded for the Ubuntu x86_64 machine will be https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz Keep the downloaded file under downloads in the workspace folder and extract the tar file

* Next is to export the PATH of the cross toolchain and paste the below command at the bottom of the /home/user/.bashrc file
```
export PATH=$PATH:/home/hjz1kor/embedded_linux/downloads/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin
source /home/user/.bashrc
```

* Now if you press arm in the terminal, you should be able to see arm-linux-gnueabihf-addr2line/c++/gcc etc...

### Compilation of U boot source tree

* Method 1: execute the script available in the u-boot source tree
```
chmod +x build-am335x.sh
sudo ./build-am335x.sh
```

* Method 2: Execute the commands manually to complire the U boot source tree

### Configuring and generating SPL,MLO and u-boot images

* Compilation steps cab be found from https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/notes/compilation_commands#L11

* distclean will delete all the previously compiled/generated object files. Don't give full name for cross compiler, it will take by itself
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- distclean
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- am335x_evm_defconfig
    HOSTCC  scripts/basic/fixdep
    HOSTCC  scripts/kconfig/conf.o
    SHIPPED scripts/kconfig/zconf.tab.c
    SHIPPED scripts/kconfig/zconf.lex.c
    SHIPPED scripts/kconfig/zconf.hash.c
    HOSTCC  scripts/kconfig/zconf.tab.o
    #
    # configuration written to .config
    #
```

* Configuration will be written to .config. We can open the file but shouldn't edit it. Distclean will delete this file

* To edit this file, run the below command to edit the settings other than default settings
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

* Next step is to compile the
```
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j4 // 4 core machine will instuct the make to spawn 4 threads
```

* U-boot will generate spl image(under spl directory), MLO(root folder of u-boot source code) and u-boot.img(root folder of u-boot source code)