# UART Mode

UART mode is enabled when S0 is unconnected or high.
UART communication is via pin 2 and 3 (CT and DR).

## Serial communications parameters

* 9600 bps (default) or 115200 bps (can be set in software)
* Parity: N
* Data bits: 8
* Stop bit: 1

## Message format (GY-33 output)

| Byte | Value | Description |
| --- | --- | --- |
| 0 | 0x5A | Header |
| 1 | 0x5A | Header |
| 2 | 0x15, 0x25, 0x45 or 0x55 | Message type |
| 3 | 0 to 0xFF | Data Length |
| 4 to (Data Length + 3) | 0 to 0xFF | Data bytes |
| Last | 0 to 0xFF | Checksum |

### Message Type

| Value | Description |
| --- | --- |
| 0x15 | Raw RGBC |
| 0x25 | Brightness, Color temperature, Color |
| 0x45 | Processed RGB |
| 0x55 | I2C Address |

### Data Length

Number of data bytes. Does not include the checksum.

### Checksum

The checksum byte is calculated by summing all the previous bytes (starting from byte 0), and taking only the lower 8 bits.

## Raw RGBC (0x15)

| Data byte | Description |
| --- | --- |
| 0 | Red (high byte) |
| 1 | Red (low byte) |
| 2 | Green (high byte) |
| 3 | Green (low byte) |
| 4 | Blue (high byte) |
| 5 | Blue (low byte) |
| 6 | Clear (high byte) |
| 7 | Clear (low byte) |

### Example

```
<5A-5A-15-08-01-78-01-92-00-4C-05-05-33 >

R = (0x01<<8) | 0x78 = 376
G = (0x01<<8) | 0x92 = 402
B = (0x00<<8) | 0x4c = 76
C = (0x05<<8) | 0x05 = 1285
```

## Brightness, Color temperature, Color (0x25)

| Data byte | Description |
| --- | --- |
| 0 | Brightness (Lux) (high byte) |
| 1 | Brightness (Lux) (low byte) |
| 2 | Color temperature (high byte) |
| 3 | Color temperature (low byte) |
| 4 | Color (high byte) |
| 5 | Color (low byte) |

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

### Example

```
<5A-5A-25-06-02-CC-0C-5D-00-02-18 >

Lux   = (0x02<<8) | 0xcc = 716 (lux)
CT    = (0x0c<<8) | 0x5d = 3162 (K)
Color = (0x00<<8) | 0x02 = 2 (yellow)
```

## Processed RGB (0x45)

| Data byte | Description |
| --- | --- |
| 0 | Red |
| 1 | Green |
| 2 | Blue |

These are 8 bits value (0 to 255), and the clear value is not represented.
It's not clear how the raw values are converted to these processed values.
If anyone manage to find out more, do share.

### Example

```
<5A-5A-45-03-FF-FF-4C-46>

R = 0xFF = 255
G = 0xFF = 255
B = 0x4C = 76
```

## I2C Address (0x55)

| Data byte | Description |
| --- | --- |
| 0 | I2C Address |

Returns the I2C address for I2C mode.
It defaults to 0x5A, but this can be changed.

### Example

```
<5A-5A-55-01-5A-64>

I2C Address = 0x5A
```

## Message format (GY-33 input)

| Byte | Value | Description |
| --- | --- | --- |
| 0 | 0xA5 or 0xAA | Header |
| 1 | Varies | Command byte. See below for details. |
| 2 | 0 to 0xFF | Checksum |

The GY-33 can be configured via UART.
See below for the different types of commands it can accept.

### Command Byte

There are 8 types of commands.

* UART Output Configuration
* LED Setting
* Save LED Setting
* Query
* Baud Rate Configuration
* White Balance Calibration
* Set I2C address
* Set Integration Time

### Checksum

The checksum byte is calculated by summing all the previous bytes (byte 0 and 1), and taking only the lower 8 bits.

## UART Output Configuration

Header = 0xA5

### Command Byte

| Bit | Meaning |
| --- | --- |
| 7 | 1: After power-on, output according to the last output configuration, 0: No automatic output |
| 6 | Must be 0 |
| 5 | Must be 0 |
| 4 | Must be 0 |
| 3 | Must be 0 |
| 2 | 1: Continuous output of Raw RGBC values, 0: No output |
| 1 | 1: Continuous output of Brightness, Color temperature, Color values, 0: No output |
| 0 | 1: Continuous output of Processed RGB values, 0: No output |

### Example

```
<0xA5+0x81+0x26>

0x81 = 0b10000001

bit 0: Continuous output of Processed RGB values.
bit 7: Save setting and restore it on power up.
```

## LED Setting

Header = 0xA5

### Command Byte

Command byte = 0x60 + LED Setting

LED Setting must be a value between 0 to 10 (0x0A); the higher the value, the lower the brightness.
When LED Setting is 10 (0x0A), the LED will be off.

This setting will not be saved on power off.
Power on default is 0x63.

### Example

```
<0xA5+0x60+0x05>

LED Setting = 0 (highest brightness)
```

## Save LED Setting

Header = 0xA5

### Command Byte

Command byte = 0xCC

Save the current LED setting and restore it on power on.

### Example

```
<0xA5+0xCC+0x71>

Save the current LED setting and restore it on power on.
```

## Query

### Header and Command Byte

| Header | Command Byte | Meaning |
| --- | --- | --- |
| 0xA5 | 0x51 | Output Raw RGBC values |
| 0xA5 | 0x52 | Output Brightness, Color temperature, Color values |
| 0xA5 | 0x54 | Output Processed RGB values |
| 0xAA | 0xF5 | Output I2C Address |

Note that for I2C address, it uses a different header (0xAA).

### Example

```
<0xA5+0x54+0xF>

Output Processed RGB values
```

## Baud Rate Configuration

Header = 0xA5

### Command Byte

Command byte = 0xAE (9600 bps) (Default)

...or...

Command byte = 0xAF (115200 bps)

### Example

```
<0xA5+0xAF+0x54>

Set baudrate to 115200 bps
```

## White Balance Calibration

Header = 0xA5

### Command Byte

Command byte = 0xBB

When this command is received, the GY-33 will perform a white balance calibration.
The calibration result will be saved, and will be reloaded upon power on.

You should place the GY-33 in front of a white object before sending this command.

### Example

```
<0xA5+0xBB+0x60>

Perform a white balance calibration
```

## Set I2C address

Header = 0xAA

Note that this command uses a different header (0xAA).

### Command Byte

Command byte = Desired I2C 7-bit Address (0 to 0x7f)

Sets the I2C address to the desired value.
Default address is 0x5A.

### Example

```
<0xAA+0x5A+04>

Sets the I2C address to 0x5A
```

## Set Integration Time

Header = 0xAA

### Command Byte

| Command Byte | Integration Time |
| --- | --- |
| 0x58 | 700ms |
| 0x59 | 154ms |
| 0x5A | 100ms (Default) |
| 0x5B | 24ms |
| 0x5C | 2.4ms |

Increasing integration time will provide a higher resolution for the raw RGBC values, but at the expense of a lower update frequency.