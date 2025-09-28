* TFTP stands for trivial file transfer protocol which can be used to transfer a file between TFTP server and TFTP client

* Connect board to host PC using Ethernet cable in the ethernet connector. Here host PC acts as TFTP server and board as TFTP client. First boot the board using SD card i.e., RAM fetches the SPL from SD card. SPL fetches u-boot.img and then u-boot.img fetches Linux kernel(uImage),dtb and initramfs from Linux host PC at /var/lib/tftpboot

* U boot is because it supports all the tftp commands to transfer the file from host PC to memory location in the target. All these procedure can be written in uEnv.txt to automate the TFTP boot process which can be kept in the SD card

### Prepare TFTP host
1. First on your Ubuntu host run the below command using your terminal program.
    $ sudo apt-get update
    $ sudo apt-get isntall
    $ sudo apt upgrade

2. install TFTP server using $ sudo apt-get install tftpd-hpa command. This command installs the tftpd deamon. tftpd is a server for the Trivial File Transfer Protocol

3. Create/Open the file “tftpd-hpa” in the below directory and put the below entry in to this file  and save it
    $ sudo vim /etc/default/tftpd-hpa

4. Add the following entries inside the file:
    TFTP_USERNAME="tftp"
    TFTP_DIRECTORY="/var/lib/tftpboot"
    TFTP_ADDRESS=":69"
    TFTP_OPTIONS="--create --secure"

5. Create a folder /var/lib/tftpboot and execute below commands
    $ sudo mkdir -p /var/lib/tftpboot
    $ sudo chown tftp:tftp /var/lib/tftpboot
    $ sudo chmod -R 777 /var/lib/tftpboot

6. Now we can start the TFTP daemon using $ sudo systemctl start tftpd-hpa

### Testing TFTP boot on board

* Copy all the file from https://github.com/niekiran/EmbeddedLinuxBBB/tree/master/pre-built-images/tftp-boot to /var/lib/tftpboot location

* Identify the ethernet interface name from ifconfig and run ifconfig enps0(ethernet interface name) ipadress(from ifconfig command)

* Power/reset the board to get into u-boot prompt. First configure "serverip" and "ipaddr" of the u-boot with the proper value
```
setenv serverip 192.168.27.1(server IP address obtained from ethernet interface in ifconfig)
setenv ipaddr 192.168.27.2(client IP address)
```

* Ping from target to host

* Now get the u-boot image from TFTP. Asking u-boot to fetch the Linux kernel image. And also set boot args in the u-boot prompt
```
tftpboot 0x82000000 uImage
tftpboot 0x88000000 am355x-boneblack.dtb
tftpboot 0x88080000 initramfs
setenv bootargs console=ttyO0,115200 root=/dev/ram0 rw initrd=0x88080000
bootm 0x82000000 0x88080000 88000000
```

* Once the boot is completed, login screen will display. The advantage of booting over TFTP is that, if there is any update in the Linux kernel image then we need not to flash it to eMMC/SD card, we can directly load the image using TFTP and check the latest boot using the image

* Whatever the file needs to be transfered, it has to be transfered to /var/lib/tftpboot because TFTP protocol will only look for this location to load the files from host. Ex: tftp -r filename -g tftp_server_ip. If the ip is not configured, configure it using ifconfig
