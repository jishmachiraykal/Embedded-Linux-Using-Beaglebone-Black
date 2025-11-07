### AM355X GPIO subsystem and expansion header

* GPIO in TRM can be found in https://www.ti.com/lit/ug/spruh73q/spruh73q.pdf?ts=1757495289153&ref_url=https%253A%252F%252Fwww.ti.com%252Fproduct%252FAM3358

* In this SOC, there are 4 GPIO modules(GPIO0, GPIO1, GPIO2, & GPIO3) and each support 32 pins

* BBB had 2 expansion headers. In P8 expansion header, there are 46 pins, PROC name tells expansion pins on the SOC(R8,T8)

* Let's say expansion header P8, ping 5 name GPIO1_8 means it is the default state of the pin like during power off, if it is in MODE0 then it is used by gpmc engine of the SOC(gpmc_ad2) and if the pin is configured for MODE1, it will be used by mmc engime(mmc1_dat2). Hence we can use single pin for different uses using pin multiplexing. This can be seen in the expansion header sec of SRM: https://elinux.org/Beagleboard:BeagleBoneBlack. In P* we can get 44 GPIO's

* Theory is same applicable to P9 expansion header as that of above. On the expansion header P9, it is 28. 44 + 28 = 67

* GPIO1[6] means 6th ping of GPIO1 as there are 32 pins for each GPIO

* In the target, GPIO's are listed under /sys/class/gpio. We can control all of them what is listed under. Here modules(0,1,2 & 3) are mentioned, it is just the pin ie., GPIO20. So there is a way to convert which is shown below

```
GPIO1[6] = 1*32 + 6 = GPIO38 //if there is any entry inside the target with gpio38, then it is 6th ping of GPIO1(module)
```

* Kernel/sw uses GPIO38 and not GPIO1[6]

### GPIO's and mode configuration registers

* There are 324 pins on the SOC and each pin can have upto 8 modes(mode0-mode7). There must be register in each pin using which we can put GPIO to different modes. Control module engine of the SOC gives dedicated registers and using that we can change the mode

* Pad(pin) configuration registers are present at offset 800h. Base_address(of the control module) + 800h(section 9.3.1.50 of TRM) = from here pad(pin) configuration reg starts

* In the memory map we can see the control module(https://www.ti.com/lit/ug/spruh73q/spruh73q.pdf?ts=1757495289153&ref_url=https%253A%252F%252Fwww.ti.com%252Fproduct%252FAM3358) = 0x44E1_0000

* 0x44E1_0000(offset) +800h= conf_gpmc_ad0(Its function given in 'Table 9-60. conf_<module>_<pin> Register Field Descriptions' of TRM)

### Exploring pin details using SYSFS entries

* In the BBB, go to sys/kernel/debug/pinctrl/0x44E1_0000.pinmux(base address of control module), here "cat pins" gives the control module register and its mode. The pin no here i.e., pin 0,1,2... will not match with naming convention of expansion header pin no i.e., P8_3,P8_2 etc.. Mapping these is a tedious process

* In the BBB, go to sys/kernel/debug/pinctrl/0x44E1_0000.pinmux(base address of control module), here "cat pinmux-pins" gives the functionality of the pins. If any pin is set to MUX UNCLAIMED then we can configure it to any mode as we want

* In the BBB, go to sys/kernel/debug/pinctrl/0x44E1_0000.pinmux(base address of control module), here "cat pingroups" gives the number of groups

* Ping 21,22,23 and 24 are used for user LED's and we can confirm it using srm and search for "user LEDs"

### Controlling user LED's using SYSFS entries

* User LED's corresponds to GPIO are GPIO1_21(1*32 + 21)= GPIO53, GPIO1_22(GPIO54),GPIO1_23(GPIO55),GPIO1_24(GPIO56)

* Go to "cd /sys/class/gpio", ls gpio53 -> no such file or dir

* We can create a entry to control this gpio by --> "echo 53 > export". Now we will get device or resouce busy because user LED gpio's 53 to 56 currently claimed these GPIO's and trying to clain it throws the error. So it is not possible to create a new entry for this

* These user LES's entry can be seen in "cd /sys/class/leds" and "cd \beaglebone:\user0:\led0". Here we can see various files using which we can control LED's

* To turnoff LED0, "cat trigger" and
```
    * echo "none" > trigger  //Now user LED0 should stop blinking
    * echo "default-on" > trigger //user LED0 gone high
    * echo "heartbeat" > trigger //user LED0 showing the sequence of heartbeat
    * echo "none" > trigger  //now cat trigger shows none inside [] because that is being triggered now
    * echo "timer" > trigger //user LED will be blinking with some delay, looks a perfect square bracket
```

* We can alter the on/off time of timer using delay_off and delay_on
```
    * echo "100" > delay_on // user LED0 will be on for 100ms
    * echo "500" > delay_off // user LED0 will be on for 500ms
```
This is repeating because trigger was set to timer in the above step

* mmc0/mmc1 can also used as trigger when we access the mmc card i.e., external mmc card(mmc0)/ internal mmc card(mmc1), we can make the LED blink
```
    * echo "mmc1" > trigger
```

* Now let us access internal emmc memory to check if user LED0 blinks, create a file "vi /home/foobar" and write "hello" inside it, we can see LED blinking now. Now save it, saving here is nothing but saving in the emmc memory. Now LED will blink again. Execute "sync" to flush everything to the filesystem. Now it'll blick and doing sync again won't blink because it would have alredy flushed the data

* Note: accessing mmc1 did not work on minicom as it did not show mmc during "cat trigger"

* Opening the file/writing something into it will make the led blink

* We can make led to blick when connected to external mmc card(mmc0) i.e., when SD card is connected

### Controlling user LED's using C application

* Get the existing projects from https://github.com/niekiran/EmbeddedLinuxBBB/tree/master/Project_Src and LED's source code from https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/Project_Src/BBB_led_control/src/BBB_led_control.c

* snprintf outputs the string to buffer, whereas printf displays the message to console

* Learn about open and write prototype in C, to get more understanding about https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/Project_Src/BBB_led_control/src/BBB_led_control.c#L41 and https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/Project_Src/BBB_led_control/src/BBB_led_control.c#L51

* sysfs is indeed a pseudo file system that presents information about various kernel subsystems, enabling user-space applications and tools to interact with kernel data in a structured way. This aligns with the learning objective of understanding system file structures and their purposes

* 6mA is the max source current of BBB expansion header pins