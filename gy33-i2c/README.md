# I2C Mode

I2C mode is enabled when S0 is connected to ground.
I2C communication is via pin 2 and 3 (CT and DR).

I2C is less capable than UART mode.
Most notably, it lacks support for setting integration time.

While it is also possible to communicate directly with the TCS3472 via pins 7 and 8 (SDA and SCL) when the S1 pin is grounded, this document is NOT for such purpose.
If you wish to read or control the TCS3472 directly, you should refer to the TCS3472 datasheet and not this document.

## I2C Settings

The default address is 0x5A.
This address can be changed via UART mode.

The signal clock needs to be between 40Khz to 200Khz.

## Registers

| Address | Description |
| --- | --- |
| 0x00 | Raw Red (high byte) |
| 0x01 | Raw Red (low byte) |
| 0x02 | Raw Green (high byte) |
| 0x03 | Raw Green (low byte) |
| 0x04 | Raw Blue (high byte) |
| 0x05 | Raw Blue (low byte) |
| 0x06 | Raw Clear (high byte) |
| 0x07 | Raw Clear (low byte) |
| 0x08 | Brightness (Lux) (high byte) |
| 0x09 | Brightness (Lux) (low byte) |
| 0x0A | Color Temperature (high byte) |
| 0x0B | Color Temperature (low byte) |
| 0x0C | Processed Red |
| 0x0D | Processed Green |
| 0x0E | Processed Blue |
| 0x0F | Color |
| 0x10 | Config |

All registers are read-only except 0x10 which is read-write.

### Color

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

It's not clear how the colors are determined, or what it means if more than one bit is set (...or if it's even possible).
If anyone manage to find out more, do share.

## Processed RGB

These are 8 bits value (0 to 255), and the clear value is not represented.
It's not clear how the raw values are converted to these processed values.
If anyone manage to find out more, do share.

## Config

| Bit | Color |
| --- | --- |
| 7 | LED Setting |
| 6 | LED Setting |
| 5 | LED Setting |
| 4 | LED Setting |
| 3 | Must be 0 |
| 2 | Must be 0 |
| 1 | Must be 0 |
| 0 | White balance calibration |

### LED Setting

LED Setting must be a value between 0 to 10 (0x0A); the higher the value, the lower the brightness.
When LED Setting is 10 (0x0A), the LED will be off.

### White balance calibration

Appears to trigger a white balance calibration.
The calibration result will be saved, and will be reloaded upon power on.

You should place the GY-33 in front of a white object before sending this command.

### Example

```python
config = 9 << 4 # Sets LED to 9 (lowest setting that is not off)
config |= 1     # White balance calibration

i2c.writeto_mem(0x5A, 0x10, bytes(config))
```
