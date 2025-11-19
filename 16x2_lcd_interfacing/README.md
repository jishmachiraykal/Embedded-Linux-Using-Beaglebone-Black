### LCD Introduction

* Pre-requisites: Connecting wires male to male, breadboard and potentiometer 100kohm/10 kohm. LCD might not come with pre-soldered wires, in that case we have to solder it

### 16x2 LCD pin details

* There are 16 pins on the panel. 15 and 16 pins are LED +(anode) and LED -(cathode) res which are used as backlight in the LCD display. LED - should be connected to gnd and LED + to 5V. Ping 7-14(DB0 to DB 14) are used to send 8 bits 7 data parallely. Let's say if we want to transfer character 8 to lcd, then its ascii val = 0x41(hex value) = 01000001 will be passed and displayed

* Data here can be user data and also lcd commands(blink, cursor off/on). Pins 3-6 are used for lcd controls. If register select pin(ping 4) is 0 then it is a lcd command and if it is 1 then it is user data. If R/W(pin 5) is 1 then reading from lcd and if 0 writing to LCD. We can read RAM Address, address counter(address of internal RAM), busy status etc... Enable pin(pin 6) is actually used to instruct latch the data. Ex: If we want to transfer A to LCD then to latch this data to instruction reg(lcd command)/data(user command) reg by making high - low transition on enable pin.

* Gnd(pin 1) and VCC(pin2) are first 2 pins

### Understanding DDRAM, CGRAM & CGROM

* HD44780U and Sitronix are some of the types of LCD controllers found

* Display data RAM is of 80 bytes, character generator RAM is of 64 bytes and character generator rom is of 9920 bits

* If A is passed to LCD controller, A = 65(ascii)= 0x41(hex value) = 01000001, then it will go to data register, then to DDR to store first character(it can store upto 80 characters) and address counter will be incremented by 1. Then DDRAM will consult CGROM to find suitable pattern(dot pattern what we see in lcd) for 'A'. CGROM contains all the patterns for all the std ASCII values/ special characters

* These patterns are stored as series of bytes, CGROM will also support for Japanese and Chinese characters

* If there is a special pattern like heart and we have to write the in series of pattern and send it to CGRAM in this case, cannot write it to CGROM because read only

### Understanding LCD commands

* The information related to commands can be seen in the data sheet of HD44780U and Sitronix. All the commands are controlled using bits/pins we saw in the above section

* clear display is the command clear entire display and sets DDRAM address to 0 in address counter. It changes the DDRAM contents

* Return home is sets the DDRAM address in address count to 0 and also returns display from being shifted to original position. DDRAM contents remains unchanged

* If I/D=1 increments the DDRAM address by 1 when a character code is written into/read from DDRAM and in I/D=0 it decrements

* When S=1, shifts entire display to right(I/D=0) or letf(I/D=1) and has no effect when S=0

* Display off/on control is used to turn off/on cursor/display and function set command is used to set interface data length, number of display lines and character font

* Set CGRAM address and DDRAM address is used to set address where we want the data to be written

* Read busy flag and address commands indicating internal operation is being performed and reads address counter contents

* Go though https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/Project_Src/Drivers/lcd/lcd_driver.h to see how these addresses are incorporated as macros and used to control LCD in the driver functions

* N=0, only one line will be activated in the LCD display and N=1, two lines will be activated

### Connecting BBB and LCD

* Connection details: https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/Project_Src/lcd_text/src/lcd_text.c#L32

* DB0-3 are not used because handling only 4 bits