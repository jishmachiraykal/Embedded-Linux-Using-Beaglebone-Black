### Ping check

* Connect BBB to host PC using TTL serial cable, USB cable and 5V power adapter

* lsusb should show USB device and ifconfig shouls also show interface with enxxxx in 192.168.7.1 IP address

* Open BBB via minicom /dev/ttyUSB0 and ifconfig. If there are no usb0 interface then execute the below command
```
sudo modprobe g_ether
sudo ip addr add 192.168.7.2/24 dev usb0
sudo ip link set usb0 up
ip addr show usb0 // should see inet 192.168.7.2/24
```

* Now from BBB, ping -c4 192.168.7.1 and from host PC 192.168.7.2

### SSH

* ssh debian@192.168.7.2

* Remove the hostgen key if needed

### SCP

* Once SSH connection is stable, execute "scp ota.py debian@192.168.7.2:/home/debian"

### STFP

* sftp debian@192.168.7.2
```
put abc.py
```

* Getting permission denied error which some paths on BBB