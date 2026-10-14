
[![Arduino CI](https://github.com/RobTillaart/MB1242/workflows/Arduino%20CI/badge.svg)](https://github.com/marketplace/actions/arduino_ci)
[![Arduino-lint](https://github.com/RobTillaart/MB1242/actions/workflows/arduino-lint.yml/badge.svg)](https://github.com/RobTillaart/MB1242/actions/workflows/arduino-lint.yml)
[![JSON check](https://github.com/RobTillaart/MB1242/actions/workflows/jsoncheck.yml/badge.svg)](https://github.com/RobTillaart/MB1242/actions/workflows/jsoncheck.yml)
[![GitHub issues](https://img.shields.io/github/issues/RobTillaart/MB1242.svg)](https://github.com/RobTillaart/MB1242/issues)

[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](https://github.com/RobTillaart/MB1242/blob/master/LICENSE)
[![GitHub release](https://img.shields.io/github/release/RobTillaart/MB1242.svg?maxAge=3600)](https://github.com/RobTillaart/MB1242/releases)
[![PlatformIO Registry](https://badges.registry.platformio.org/packages/robtillaart/library/MB1242.svg)](https://registry.platformio.org/libraries/robtillaart/MB1242)


# MB1242

Arduino library for the MaxBotic I2C MB1242 distance sensor.


## Description

**Experimental - work in progress**

This library is to use the MB1242 and compatible distance sensors.

Note: not tested with hardware yet.

Came across this sensor on the Arduino forum and to understand it better I tried to write an Arduino library for it.
So consider it work in progress.
If you have hardware, please test and provide feedback so the library
can mature.

TODO elaborate

Voltage 3.0 - 5.5 V, so should work for ESP32 and AVR.

Temperature -40° - 65°

Sample rate up to 40 Hz (need status pin).

Device is operational after ~115 ms


Feedback as always is welcome.


### Correction for speed of sound.

The library or sensor does not compensate the speed of sound for temperature.
Look at the SRF05 library for a function to calculate the speed of sound.
To be added in the future.


### Datasheet warning

_Do not apply this product to safety protection devices or emergency stop equipment,
and any other applications that may cause personal injury due to the product's failure._


### Hardware

```
             TOPVIEW MB1242
         +--------------------+
   pin 7 | o  GND             |
   pin 6 | o  VCC             |
   pin 5 | o  SCL             |
   pin 4 | o  SDA             |
   pin 3 | o   x              |
   pin 2 | o   x              |
   pin 1 | o   x              |
         +--------------------+
```

|  Pin  |  description  |
|:-----:|:--------------|
|   7   |  Ground 
|   6   |  VCC
|   5   |  SCL - I2C clock
|   4   |  SDA - I2C data
|   3   |  not used
|   2   |  status + not implemented - address announce
|   1   |  not implemented - address used


Pin 2 is HIGH during distance measurement, and LOW when IDLE.
During power up pin 2 "sends" a high pulse indicating its 8 bit address.
100 us per unit, so address 224 gives a pulse of 22.400 micros.
One can measure this with digitalRead() or pulseIn() only if you can 
control the startup. The library does not support this function.

Using the status pin can speed up the detection that the device is ready,
and thus the reading of the device.

If pin 1 is HIGH or unconnected it will use the internal I2C address set
with changeAddress(). If pulled LOW it will use factory default address.
The library does not support this function.


### Compatibles

Datasheet (ds) mentions:

|  device  |  min range   |  ds  |   beam   |notes  |
|:---------|:------------:|:----:|:--------:|:------|
|  MB1202  |  20 cm / 6"  |  09  |  wide    |  people, high sensitivity
|  MB1212  |  20 cm / 6"  |  10  |  medium  |  best balance
|  MB1222  |  25 cm / 6"  |  11  |  medium  |  best balance
|  MB1232  |  25 cm / 6"  |  12  |  narrow  |  large targets
|  MB1242  |  25 cm / 6"  |  13  |  narrow  |  very large targets


### Related

Distance sensors
- https://github.com/RobTillaart/A02YYUW - Serial distance sensor.
- https://github.com/RobTillaart/MB1242 - this lib.
- https://github.com/RobTillaart/SRF05 - distance sensor
- https://github.com/RobTillaart/US500 - US500 + US4000 underwater distance sensor.

Other
- https://github.com/RobTillaart/lengthConverter
- https://github.com/RobTillaart/printHelpers - print inches and feet
- https://github.com/stoduk/PingSerial  Serial IO with distance sensors e.g. US-100


### Tested

TODO: Test on Arduino UNO and ESP32


## I2C

### I2C Address

The default address is 0x70 (112).
The address can be changed with **changeAddress(uint8_t addr)**.
Invalid addresses reported in the datasheet are 0, 80, 164, 170.
ChangeAddress() will return false if these are used.

It is not clear if a restart is needed when the address is changed.

To be tested.


### I2C multiplexing

Sometimes you need to control more devices than possible with the default
address range the device provides.
This is possible with an I2C multiplexer e.g. TCA9548 which creates up
to eight channels (think of it as I2C subnets) which can use the complete
address range of the device.

Drawback of using a multiplexer is that it takes more administration in
your code e.g. which device is on which channel.
This will slow down the access, which must be taken into account when
deciding which devices are on which channel.
Also note that switching between channels will slow down other devices
too if they are behind the multiplexer.

- https://github.com/RobTillaart/TCA9548


### I2C Performance

Datasheet states it only works at 20 - 100 kHz.

To be tested.


## Interface

```cpp
#include "MB1242.h"
```

### Constructor

- **MB1242(uint8_t address, TwoWire \*wire = &Wire)** optional select I2C bus.
- **bool begin()** resets internal variables and checks if device address is 
visible on the I2C bus.
- **bool isConnected()** Checks if device address can be found on I2C bus.
- **uint8_t getAddress()** Returns the device address, default 0x70 (112).


### Async Request

- **bool requestDistance()** request a new distance measurement. 
This might take up to 100 ms.
- **void setStatusPin(uint8_t statusPin)** option to connect pin 2
to the library so ready() can detect measurement ready much faster.
It is possible to use an IRQ pin as status pin and set a flag
in the IRQ. 
- **bool ready()** returns true, up to 100 ms after a request was made.
Will return false if no request was made.
If the statusPin is set, it will read the status pin instead of 
waiting hard for 100 millis.


### Async Read

- **uint16_t readCM()** read the measurement from the device.
Returns range in centimeter.
- **float readMeter()** wrapper around readCM(), converts to meters.
- **float readInch()** wrapper around readCM(), converts to inches.
- **float readFeet()** wrapper around readCM(), converts to feet.
Note decimal part are 'decimals', 0.5 feet == 6 inch.
- **uint32_t lastRead()** returns timestamp of last read.

One can use the printHelpers library to print inch and feet nicely.


### Address

- **bool changeAddress(uint8_t address)** changes the address of the device.
Invalid addresses 0, 80, 164, 170 are checked and return false.


### Debug

- **uint8_t getLastError()** returns last error of low level communication.


## Future

#### Must

- improve documentation
- get hardware to test

#### Should

- implement derived classes (wrappers?)
- SOS calibration for temperature and humidity.
- improve error handling

#### Could

- add examples
  - IRQ for status pin.
- add unit tests (if possible)

#### Wont


## Support

If you appreciate my libraries, you can support the development and maintenance.
Improve the quality of the libraries by providing issues and Pull Requests, or
donate through PayPal or GitHub sponsors.

Thank you,


