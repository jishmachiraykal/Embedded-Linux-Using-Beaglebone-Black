* BBB comes with in-build 4GB eMMC memory. eMMC is connected to mmc1 interface. By default BBB will boot from mmc1 interface when powered up

* We need to re-flash the eMMC memory to boot from eMMC by the help of SD card. Process is as follows:
    1. Download the latest debian OS
    2. Write the bootable image to the SD card
    3. Boot the board from SD card
    4. Execute emmc flasher script present in the SD card
    5. Flasher script will flash all the contents from SD card memory to eMMMC memory

* Flashing takes 5-10 minutes. Once it is done SD card can be removed from board and BBB will boot from eMMC. This has to be done whenever beaglebone.org releases new version

* Flasher script will divide the eMMC memory into two partitions called BOOT and ROOTFS. Then it will format the partitions to create file system(FAT & EXT4). Then copy the contents of SD card into newly created partions

* Check the debian OS image using lsb-release -da and latest version will be in https://www.beagleboard.org/distros

* Flash OS images to SD cards & USB drives, safely and easily can be done using installing https://etcher.balena.io/#download-etcher software and download for x64 software. Download both debian and balena images and place it in workspace downloads folder

* Launch the balena ethcher application and select the debian file downloaded. In the target section select the SD card generic device. Then flash the image, it might take around 10-15 minutes

* Now power off the board and boot from SD card usinf S2 button. Once the boot is over, we can see devive enumenration in the PC i.e., we can see the device connected to the drive in the file explorer

* The board will be booted from SD card, now the contents has to written to eMMC. To do that go to /opt/script/tools/eMMC/opt/scripts/tools/eMMC. Run init-eMMC-flasher-v3.sh. Flashing will start and LED pattern on BBB will change now. After flashing, board will go to power down mode. LEDs are off and remove the SD card and press power button board will boot from eMMC

* Connect to board via ssh or minicom

* We can also connect to board using web interface when debian is connected by 192.168.6.2 in browser. Cloud9 will open, go beaglebone/black folder few Java scripts will open to toggle LED, blink LED etc... Run the script directly in the cloud9 console and see the result in the board

* ifconfig lists all the devices, eth0 is ethernet device. Usb0/1 are usb devices to comminicate over ethernet. If will be configured automatically, debian will configure the IP statically

* Once you connect the board, go to settings under network you'll will be able to see 2 new ethernet connection with same subnet IP address as ifconfig what we see inside target. Also in the host PC if you do ifconfig, you'll be able to see the configuration.If you remove the hw these two will go off. This internet connection will be done automatically

* enp0s* interface in the host PC is for ethernet connectivity

* Now in order to enable the internet connection in BBB, open /etc/resolve.conf file. Create one if it is not present and add below DNS addresses
```
    nameserver 8.8.8.8
    nameserver 8.8.4.4
```

* Next add default gateway address to routing table as "route add default gw 192.168.7.1(host PC IP address) usb0" and ""route add default gw 192.168.6.1(host PC IP address) usb1"

* Ethernet interface can be obtained by doing ifconfig and look for e*** interface where it has the same subnet IP address of target

* Now in the host side share the internet between wi-fi and ethernet. To do that first do ip forwarding by "echo 1 > /proc/sys/net/ipv4/ip_forward". Next do ip table settings i.e., find the wifi interface by running ip link and interface name starts with wl. IP table commands are:
```
   iptables --table nat --apend POSTROUTING --out-interface <wi-fi interface> -j MASQUERADE
   iptables --apend FORWARD --in-interface <ethernet interface to share with> -j ACCEPT
```

* Now ping www.google.com from target and it should work

* Get the interface device type using "nmcli --get-values GENERAL.DEVICE,GENERAL.TYPE device show"