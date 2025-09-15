* Linux device tree is also called as flattened device tree

* The on-board peripherals like Serial flash, LEDs, buttons, SD card connector, USB connector etc.. which connect to SPI, ethernet, I2C, SDIO have no capability to announce their existence on the board by the themselves to the OS. But USB has in-built intelligence to send its details to the OS by pushing some information(it supports dynamic discoveribility). All these device are called flat-form devices

* In order to make Linux kernel know about these platform devices is by hardcoding these value in a config file called board file. Whenever you modify the file to add new device, you need to re-compile the kernel

* When a driver for a particular platform is loaded, Linux calls the probe function of the driver if there is any match in its platform device database.In the probe function of the driver, you can do devive initialization

* Each board will have its down board_config.c/any config file, which means every board has its own kernel image. Therefore, config file of one board may/may not work in another board

* Hence, Linux commiity wanted to cut off this limitation i.e., platform device specific details in to the Linux kernel. Then arm community came up with device tree idea where every vendor should come up with device tree source(DTS) file

* This file contains all the details of the board written using pre-defined syntaxes which has all the details of the hw. Every board will have dts file which will be compiled using special kind of compiler to convert it into DTB(stream of bytes/binaries)

* When DTS file is edited to add the new entry, kernel needs to be compiled again. Need to complie the DTS to obtain the DTB

* When the kernel boots, we need to tell where DTS resides. With this approach, we need to change the kernel when board changes, just need to change the DTS file

* If i connect DDR3 RAM to my laptop's pci connector ? Can i say DDR3 memory device is a platform device. No because it is connected to PCI bus which is self discoverable. Platform devices are devices which are not self discoverable