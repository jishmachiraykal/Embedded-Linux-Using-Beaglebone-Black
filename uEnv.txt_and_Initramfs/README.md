* Reset the board, so the board will boot from eMMC and halt at u-boot so that u-boot prompt will open. In the u-boot command prompt type help it will show all the operations it performs

* If u-boot prompt is not working, then hold the space bar and hit the reset button. u-boot prompt will be entered and type boot to go back to normal boot prompt

* To understand what each command does, type help sleep for example. printenv command shows all the environment variables along with its value

* Similar to Linux environment variables, u-boot also set to standard as well as user-defined environment variables which can be used to override or change the behaviour of the u-boot

* And to see what value the environment variable has, type printenv soc this will give the SOC which is present on the board. setenv is the command to set new environment variable and assign value to it or to modify the existing values as well. Ex: setenv serverip 192.168.27.1

* Here boot command runs an env variable(printenv) bootcmd. bootcmd runs set of commands like "bootcmd=if test ${boot_fit} -eq 1; then run update_to_fit; fi; run findfdt; run init_console; run envboot; run distro_bootcmd". If bootcmd is changed, then boot behaviour will be changed
```
help boot
boot - boot default, i.e., run 'bootcmd'

Usage:
boot
```

* load <interface> [<dev[:part]> [<addr> [<filename> [bytes [pos]]]]], here interface can be emmc,usb etc... dev is the device number(can be checked using mmc list in u-boot prompt). Part is partition number(1st/2nd), addr is the address of DDR memory where image will be transfered, filename is binary filename and [bytes [pos]] is size of the binary file. Ex: "load mmc 0:2 0x82000000 uImage". Here 0 is SD card, 2 is 2nd partition where BOOT is present and uIMage is the image in the BOOT partition and last parameter binary file size is ignored here

* Next linux device tree needs to be loaded, for that run, "load mmc 0:2 0x88000000 am335x-boneblack.dtb". am335x-boneblack.dtb is the name of the device tree binary from BOOT partition and 0x88000000 is the address of the DTB. You should get output something like xyz bytes read in x sec after running these commands

* BBB used UART0 as serial debug terminal which is /dev/ttyO0, otherwise bootlogs will be display during boot from u-boot to Linux kernel. Fot this bootargs env variable can be used to send args to Linux kernel. Define it if not defined by executing "setenv bootargs console=ttyO0,115200". Now run load command given above once again after setting the env variable

* Now try to boot using "bootm 0x82000000 - 88000000". Boot will fail here again because Linux has no idea from where exactly it should mount the filesystem. So we have to send the location and type of filesystem using bootargs. Edit the bootargs command by "setenv bootargs ttyO0,115200 root=/dev/mmcblk0p1 rw" root value can be obtained from lsblk command i.e., rootfs partition and rw is read write access. And repeat all the load commands after setting and boot once again using "bootm 0x82000000 - 88000000"

* All the required commands are:
    1. load mmc 0:2 0x82000000 uImage
    2. load mmc 0:2 0x88000000 am335x-boneblack.dtb
    3. setenv bootargs console=ttyO0,115200 root=/dev/mmcblk0p1 rw
    4. bootm 0x82000000 - 88000000

* boot command in u-boot prompt does all these command together when executed. This can be wrapped in a uEnv.txt file.

```
help load
load - load binary file from a filesystem

Usage:
load <interface> [<dev[:part]> [<addr> [<filename> [bytes [pos]]]]]
    - Load binary file 'filename' from partition 'part' on device
       type 'interface' instance 'dev' to address 'addr' in memory.
      'bytes' gives the size to load in bytes.
      If 'bytes' is 0 or omitted, the file is read until the end.
      'pos' gives the file byte position to start reading from.
      If 'pos' is 0 or omitted, the file is read from the start.
```

* Next task is to create uEnv.txt file and transfer it to BBB. For the create a folder uEnv in workspace and in uEnv.txt add "myip=setenv serverip 192.168.1.2"/"myip=192.168.1.2" and leave one new line at the EOF. Now let's transfer the minimal uEnv.txt from host to target using serial port transfer protocols like xmodem and ymodem. For then go to u-boot prompt, execute loady and same needs to be enabled in host PC also for that press ctrl+a+s and select ymodem, go to the directory where uEnv.txt is present and hit space bar 2 times. Now to go uEnv.txt and press space bar one time to select it and hit enter.

* There might be failure in the file transfer if you leave the loady for long time. Output will be
```
loady
## Ready for binary (ymodem) download to 0x82000000 at 115200 bps...
C/0(CAN) packets, 4 retries
## Total Size      = 0x0000001d = 29 Bytes
```

* Import the downloaded uEnv.txt from memory using "emv import -t 0x82000000 29" using 0x82000000 and 29 from the results of loady. Now execute printenv myip, it will output the ip address. If you try to set senenv serverip 192.168.1.2 alone i.e., with custom variable(myip), changes will not work. Now "run myip" and then printenv serverip. serverip env value will be set

* Now add more boot commands like load mmc, setenv bootargs in the uEnv.txt file
```
    cat uEnv.txt
    myip=setenv serverip 192.168.1.2
    bootargs=console=ttyO0,115200 root=/dev/mmcblk0p1 rw
    bootcmd=echo"Booting from memory";load mmc 0:2 0x82000000 uImage;load mmc 0:2 0x88000000 am335x-boneblack.dtb;bootm 0x82000000 - 88000000;
```

* Again transfer file from host to target using ymodem and again import the downloaded file from memor/. Verify bootargs and bootcmd are set. Then execute boot in the u-boot it will use the new env variables from uEnv.txt and boot the image. You should get kernel image

* In order to boot the image using custom uEnv.txt again, once after boot we need to transfer the file again and import the downloaded file