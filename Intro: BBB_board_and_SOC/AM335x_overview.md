* SOC containing lot of peripherals and controllers like DMA controller, SPI controller, Ethernet controller, Flash memory, RAM, USB controller

* More details about A335x in https://www.ti.com/product/AM3358

* This SOC is placed in the middle of the board with partnumber starting with A3358BZCZ100

* Using SOC we can mount any controller without any problem because those controller will already be there on the SOC

* Heart of this SOC is MPU(mirco processor unit) made of ARM cortex A8 processor which is 32 bit risk processor and can run upto the speed of 1GHz

* L3 and L4 are interconnect like highways, which carry data from various peripherals to controllers and vice-versa

* There is a separate section for interconnects in the TI's manual and can have a look for more information. These are high speed data buses and using its registers we can control the data speed

* Wrt memory interfaces, it supports NAND,NOR and DDR memory interfaces. DDR memory component number is U12 and eMMC is U13, check for this number in the technical reference manual to find partnumber. Once its find get more info about partnumber to understand its data sheet and how it is connected to BBB's SOC. Here eMMC is connected to SOC using MMC interface, MMC0 is for SD card and MMC1 for e-MMC on the BBB

* This SOC doesn't come with the DSP co-processor. AM3358 is made up of 1 core ARM cortex A8 processor. ROM size of AM3359 SOC is 176 KB. GPMC  of the AM335X SOC used for interfacing of external NAND,NOR and SRAM. EMIF of the AM335X SOC used for interfaccing of external dynamic RAM(DDR)

* A single 4KB EEPROM is proveded on I2C0 that holds the board info. This info includes board name, serial number and revision information