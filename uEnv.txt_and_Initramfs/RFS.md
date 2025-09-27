* Root file system, as the name indicates, it’s a file system which Linux mounts to the "/" (root). File system is nothing but a collection of files organized in standard folder structure. There is a standard for the Linux file system. That is called "File system Hierarchy Standard". Reference for FHS https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard#cite_note-2

* RFS directory description
```
    bin     Essential command binaries
    boot    Static files of the boot loader
    dev     Device files
    etc     Host-specific system configuration
    lib     Essential shared libraries and kernel modules
    media   Mount point for removable media
    mnt     Mount point for mounting a file system temporarily
    opt     Add-on application software packages
    run     Data relevant to running processes
    sbin    Essential system binaries
    srv     Data for services provided by this system
    tmp     Temporary files
    usr     Secondary hierarchy
    var     Variable data
```

* bin/:You don’t need privileges from your System admin to execute these commands neither you need root access. Remember that this folder will not contain binaries for all the Linux commands. There is a restriction on what types of commands have to be placed in this directory, because these binaries can be executed by the common user. You can see that commands related to "repairing", "recovering", "restoring", "network configuration" ,”modules install remove”  are not found in this directory.

* boot/: This directory contains the boot related files, which are required to boot the Linux. This directory may be read by the boot loader to read the boot images like Linux kernel image, dtb, vmLinux, initramfs, etc. So this directory may be accessed by boot loader even before the kernel boots and mounts the file system

* dev/: If you want to access any i/o, networking devices, memory devices, serial devices, parallel devices, input output devices such as keyboard, mouse, display, everything will be treated like a file.So, this directory will have the file entry for every device. For example: the i2c devices may have a file entries like this
```
    /dev/i2c-0
    /dev/i2c-1
```

* etc/: This is the place where all the start-up scripts, networking scripts, scripts to start and stop networking protocols like NFS, networking configuration files,  different run level scripts will be stored

* lib/: To store the Essential shared libraries (.so.*) for dynamic linking. For example, 'C' shared library(libc), math library, python libray, etc and also the dynamically loadable kernel modules (later you will see, when we compile the kernel modules and when we run “modules install” command , all the kernel modules will go and sit in this directory under the sub directory "modules")

* media/: This is the mount point for the removable media like your USB flash drive, SD cards, camera, cell phone memory, etc. For example, when i connect my SD card to the PC, there will be 2 device files will be created for each partition i.e., /dev/sdb1 and /dev/sdb2 and theses 2 device files are automatically mounted under the /media directory. So that i can access those 2 partitions just like folders

* mnt/: This is the place where you can mount the temporary file system

* opt/: "opt" stands for "optional". This directory will be used when you install any software packages for your Linux distribution. For example if i run the command apt-get install <some packages name> then the package will be installed in this directory

* sbin/: The commands which come in the category of system administration will be stored in this directory, which is used by your sys admins for the purpose of networking configurations, repairing, restoring and recovering

* home/: The /home directory contains a home folder for each user. Each user only has write access to their own home folder and must obtain elevated permissions (become the root user) to modify other files on the system. This directory will be used to store personal data of the user

* srv/: SRV stands for "Service". The /srv directory contains “data for services provided by the system.” If you are using the Apache HTTP server to serve a website, you’d likely store your website’s files in a directory inside the /srv directory

* tmp/: Applications store temporary files in the /tmp directory

* usr/: According to FHS, it’s a "secondary hierarchy"
