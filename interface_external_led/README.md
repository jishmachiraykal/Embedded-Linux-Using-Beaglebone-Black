* cat sys/kernel/debug/gpio tells the available GPIOs in the expansion header

* car sys/kernel/debug/pinctrl/44e10800.pinmux/pins to see the state of the GPIO pins. Selecting P8_16 to connect external LED as it is in GPIO mode. To check if it is in GPIO mode follow the steps from sec 16

* Connect -ve terminal of led to gnd(P9_1/P9_2) and +ve terminal to resistor(330 /470 Ohm) and from there to P8_16 expansion header because source current(6-8 mA) coming from P8_16. We can directly connect it because here small current is flowing but it is a good practice to connect resistor in between always. If more current comes then LED might burn

* P8_16 is GPIO46, need to control from sys/class/gpio46/. If GPIO46 is not present "echo 46 > export'. "cat direction", shows if GPIO is in input mode

* Since we are driving LED now make "echo out > direction". Now perform "echo 1 > value", LED will turn ON and "echo 0 > value", LED will turn OFF

* We can also change the definition(0 for on and 1 for off) using active low, "echo 1 > activelow" Then the above above to turn off/on will work in reverse direction