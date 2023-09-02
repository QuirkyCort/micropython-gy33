# I2C Mode

I2C mode is enabled when S0 is connected to ground.
I2C communication is via pin 2 (CT/SCL) and 3 (DR/SDA).

For details of the GY-33 I2C protocol, refer to this https://github.com/QuirkyCort/micropython-gy33/blob/main/gy33-i2c/gy33_i2c_protocol.md

# Using the driver

## Constructor

### class gy33_i2c.GY33_I2C(i2c, addr=90)

* i2c: An I2C object
* addr: Address of the I2C object

## Methods

### GY33_I2C.read_all()

Returns a tuple containing the "Raw Red, Raw Green, Raw Blue, Clear, Lux, Color Temperature, Red, Green, Blue, Color" values.

The "Color" value should be interpreted as follows:

| Bit | Color |
| --- | --- |
| 7 | Blue |
| 6 | Navy Blue |
| 5 | Green |
| 4 | Black |
| 3 | White |
| 2 | Pink |
| 1 | Yellow |
| 0 | Red |

Note that only the Raw and Clear values are from the TCS3472 sensor, while the other values are derived from the raw values by the built-in micro-controller.

### GY33_I2C.read_raw()

Returns a tuple containing only the "Raw Red, Raw Green, Raw Blue, Clear" values.

This saves a few reads if you don't need the processed values.

### GY33_I2C.read_calibrated()

Returns a tuple containing only the calibrated Red, Green, Blue, and Clear values.
Calibrated values are norminally between 0 to 255, but if exposed to light level that exceeds the calibration, it may return values beyond 0 to 255.

These uses calibration values stored in the gy33 object, and are not the same as the values obtained from "read_all()" (...which uses calibration stored in the sensor module).

### GY33_I2C.calibrate_white()

Performs white calibration.
The sensor should first be placed on a suitable white surface.

After calibration, the same surface should return 255 for all RGBC values when performing a "read_cal()".

### GY33_I2C.calibrate_black()

Performs black calibration.
The sensor should first be placed on a suitable black surface.

After calibration, the same surface should return 0 for all RGBC values when performing a "read_cal()".

### GY33_I2C.set_led(pwr=0)

Sets the LED power (0 to 10).
0 will turn the LED off, while 10 will turn it to max power.

### GY33_I2C.calibrate_white_balance()

Performs a white balance calibration.
The sensor should first be placed on a suitable white surface.

This is handled by the built-in micro-controller affects the last 4 values (Red, Green, Blue, Color) of "read_all()".
The built-in microcontroller does not provide a black calibration.

## Examples

```python
import machine
import gy33_i2c
import time

i2c = machine.I2C(0, freq=100000)
gy33 = gy33_i2c.GY33_I2C(i2c, addr=90)

gy33.set_led_power(10) # Full power

while True:
    print(gy33.read_cal())
    time.sleep(1)
```