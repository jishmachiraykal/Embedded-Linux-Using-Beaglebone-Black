* Source code of the course hosted in https://github.com/niekiran/EmbeddedLinuxBBB

* BBB uses TI’s AM355x SOC, which can run at 1GHZ clock speed

* The heart of the SOC is a processor.For example AM355x SOC is powered by ARM cortex A8 processor

* Technical Reference manual of this SOC can be found in http://www.ti.com/lit/ug/spruh73p/spruh73p.pdf

* The board has 4GB of eMMC(embedded Multi Media Controller) memory chip, This is an on-board  memory chip that holds up to 4GB of data in BBB Rev C

* BBB user manual: http://elinux.org/Beagleboard:BeagleBoneBlack

* And always remember when you connect  “USB to Serial Convertor”(UART cable) to any hardware like BBB, the TX pin of this module should go to the RX pin of the another board, in this case BBB

* Best serial monitoring sw for ubuntu is minicom and for windows is terra-term, putty and hyper-terminal

* sudo apt-get update and sudo apt-get install minicom to install minicom sw in Ubuntu

* When USB to serial converter is connected, you can do dmesg in terminal and see which tty port it has connected and other details like shown below:
```
    usbcore: registered new interface driver usbserial_generic
    usbserial: USB Serial support registered for generic
    usbcore: registered new interface driver ftdi_sio
    usbserial: USB Serial support registered for FTDI USB Serial Device
    ftdi_sio 2-2:1.0: FTDI USB Serial Device converter detected
    usb 2-2: Detected FT232RL
    sisevt: FTrace sisevt_hook_syscalls_64 ret(0)
    usb 2-2: FTDI USB Serial Device converter now attached to ttyUSB0
```

* When using the minicom, if something is not working then first check serial port setup. For that ctrl A-Z, option O go to serial port setup, change hw control flow and sw control flow to NO if it is set to YES because BBB does not use any serial

* Baud Rate option in serial port setup ex: Bps/Par/Bits       : 115200 8N1 means baud rate is 115200, No parity and stop bit is 1

* save as dfl option in minicom sets the default value so that we need not to configure evrytime the same data

* After opening the minicom, run ifconfig to get the interfaces and check the IP address of USB0 or any port connected. Try to ping the IP address from another host terminal, n/w should be reachable

* ssh to target using ssh -l debian IP_address

* BBB comes with debian flavor of Linux OS.

* If windows is used we need to intall the drivers in E:\Drivers\Windows to enable us to use BBB's internet over USB

* Command to check which version of image is flashed:
```
    lsb_release -a
    No LSB modules are available.
    Distributor ID: Debian
    Description:    Debian GNU/Linux 11 (bullseye)
    Release:        11
    Codename:       bullseye
```

* We can connect to BBB hw using 192.168.7.2 in browser

* AWS cloud9 is IDE that enabled write, run and debug code by just using browser

* BBB is an example of SBC. Sitara series of TI soc is used in beaglebone. AM335x SOC hasn't got 4GB on chip eMMC memory.eMMC usually off the SOC. There is no NAND flash in BBB, it has got eMMC. DDR3 is a RAM memory of type SDRAM

* To share Host PC's internet on the board over USB (internet over USB) the board must not first support Ethernet MAC and PHY along with USB. This functionality is supported as separate USB class