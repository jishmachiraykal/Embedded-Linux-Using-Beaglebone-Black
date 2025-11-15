### Why accelerometers are used

* Accelerometer is a sensor which can measure the acceleration forces which are acted on it. Force is directly proportional to acceleration. If MPU-6050 sensor which is kept on a table with no force should ideally show the acceleration as 0, but force due to gravity "acceleration=1 g (9.8m/s2)

* Based on the raw reading from the sensor we can convert it to its equivalent 'g' values

* Accelerometers uses tilt sensing application in phones and gaming applications and to know the orientation of the body by taking earth gravity as a reference

### Why Gyroscope sensors are used

* Accelerometers measures the acceleration forces exerted on x,y,z axes and gyroscope measures the rotational movement of an object over x,y,z axes. So running on a road, will not show any reading on gyroscope. Here rotation will be measured in deg/sec

* Here gyroscope will not detect any vibration forces and just calculates the rotation which makes it ideal in many application where we cannot use accelerometer which cannot differentiate between rotation and vibrations

### MPU6050 intro

* MPU6050 is a single sensor consisting of accelerometer and gyroscope in a single IC. MPU6050 data sheet: https://invensense.tdk.com/wp-content/uploads/2015/02/MPU-6000-Datasheet1.pdf

* MPU6050 register maps: https://cdn.sparkfun.com/datasheets/Sensors/Accelerometers/RM-MPU-6000A.pdf

* Slave address of sensor is 7 bit(can be found under I2C Interface in datasheet)

* If AD0=LOW, then address=b1101000=0x68

* If AD0=HIGH, then address=b1101001=0x69

* In the breakout board AD0 is grounded, so default value=0x68

### Understanding MPU6050 accelerometer full scale range

* Refer section 6.2 Accelerometer Specifications in data sheet: https://invensense.tdk.com/wp-content/uploads/2015/02/MPU-6000-Datasheet1.pdf, Full-Scale Range -> AFS_SEL=0 -> ±2 which gives better resolution and sensitivity. Eg: Speed indicator in vehicle. 1g = 9.8 m/s2

### Converting raw acc value into g value

* If raw value on z axis = 6000, then we have to check in which full scale axis it is operating in let's say AFS_SEL=2, where value will be 4092 for every g(can be see in data sheet, section 6.2 Accelerometer Specification) = 6000/4096 = 1.46 = 1.4g

### Understanding MPU6050 gyroscope full scale range

* Refer section 6.1 Gyroscope Specifications in data sheet: https://invensense.tdk.com/wp-content/uploads/2015/02/MPU-6000-Datasheet1.pdf

* Sensor is rotating at 1 deg/sec on y axis and sensor is operating at FS_SEL=0, the raw value expected at y axis is 131
```
rotation_in_deg_y_axis= raw_value_y/131

Here 131/131= 1 deg/sec
```

### MPU6050 breakout board details

* It has 8 pins, first ping(Vcc) has to be connected to 5V, second ping to gnd and SDA and SCL pins to connected to SDA and SCL pins of BBB respectively. ADO is already pulled down by 4.7k rissitor. XDA and XCL(external) are used to interface sensor which is based on I2C. Eg: interfacing magnetometer to XDA and XCL. INT pin is used to interrupt uc using the sensor

* Connections
```
P9.4(3.3 V) = VCC
P9.1 = GND
P9_19 = SDA
P9_20 = SCL

Other pins are no connection at this time
```

### Deciding BBB I2C pins for sensor interfacing

* MPU6050 sensor communicates with I2C. There are 3 I2C controllers available for use in AM355x. In target under /dev we can see the 3 i2c device files to connect/communicate with these 3 controllers

* To know in which I2C controller we have to connect the sensor, we have to check expansion header. There are no I2C ping in P8 expansion. Pin 17 & 18 of P9 expansion header is connected to I2C1, 19 & 20 are used for I2C2. These pins will acts are I2C only when set to MODE2(17 & 18)/MODE3(19 & 20)

* P9_17 and P9_18 are pins 87 and 86 in sw/kernel and P9_19 and P9_20 are 95 &  in sw/kernel images. To check if it is in I2C mode, "cat /sys/kernel/debug/pinctrl/44e10800.pinmux/pins" 87 and 86 are set to 7(last 7 digits) which acts as GPIOs, we this cannot be used unless we change it using device tree or other means. 95 is in MODE3, so we can connect MPU6050 to I2C2 controller