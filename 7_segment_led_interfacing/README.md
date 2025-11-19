* It has 7 segments and one decimal point dedicated to a LED.So there are 8 LEDs in total This can be anode or cathode display

* In common cathode(-ve) display, cathode of all LED's are tied to GND and LED's anode can be controlled using microcontroller. In the common anode display all LED's anode are tied to +Vcc.

* There are 10 pins in 7 segment display, 7/10 pins are dedicated for segments, other 2 are Gnd/Vcc and 1 is decimal point

* In the common cathode display connect GND ping to display to GND and other 8 pins to uc through resistor to avoid burining of LED due to large current. If you make any GPIO pin(P8_14/16/12) high(pull it) then LED will glow, otherwise it turns OFF

* Here using resistor is optional because bbb will only source 6-8 mA. For the exact connection check sec 18 lecture 105

### 7 segment display up-down counter implementation

* Refer from https://github.com/niekiran/EmbeddedLinuxBBB/blob/master/Project_Src/counter_7seg/src/counter_7seg.c

* If the LED forward voltage is 2V and if 9V battery is used to driver the LED, then Whats the min value of the current limiting resister should be used to protect the LED? LED max forward current 12.5ma
```
9-2/12.5 = 0.56 = 0.56x100 = 560 Ohm
```

