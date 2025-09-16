* Reset the board, so the board will boot from eMMC and halt at u-boot so that u-boot prompt will open. In the u-boot command prompt type help it will show all the operations it performs

* To understand what each command does, type help sleep for example. printenv command shows all the environment variables along with its value

* Similar to Linux environment variables, u-boot also set to standard as well as user-defined environment variables which can be used to override or change the behaviour of the u-boot

* And to see what value the environment variable has, type printenv soc this will give the SOC which is present on the board. setenv is the command to set new environment variable and assign value to it or to modify the existing values as well. Ex: setenv serverip 192.168.27.1