# UART Mode

UART mode is enabled when S0 is unconnected or high.
UART communication is via pin 2 and 3 (CT and DR).

For details of the GY-33 I2C protocol, refer to this https://github.com/QuirkyCort/micropython-gy33/blob/main/gy33-uart/gy33_uart_protocol.md

## Serial communications parameters

* 9600 bps (default) or 115200 bps (can be set in software)
* Parity: N
* Data bits: 8
* Stop bit: 1

# Using the driver

## Constructor

### class gy33_uart.GY33_UART(uart)

* uart: A uart object

## Methods

### GY33_UART.read(wait=False, timeout=1000)

Reads from the uart, parse the messages, and store the results.
This method does not return any values; you'll need to use the corresponding "get" methods to retrieve them.

* wait: If True, the method will block until it successfully reads a single message or timeout is reached.
* timeout: Timeout in milliseconds. Only used if wait is True.

### GY33_UART.set_output(raw, lcc, processed)

Sets the values that the GY-33 will continuously output.

* raw: If True, continuously output the raw values.
* lcc: If True, continuously output the Lux (Brightness), Color Temperature, and Color values.
* processed: If true, continuously output the processed color values.

Note that you will still need to perform a "read" and a "get" to retrieve the values.

This setting will persist across a power cycle.

### GY33_UART.set_led(pwr, save)

Sets the LED power (0 to 10).
0 will turn the LED off, while 10 will turn it to max power.

* pwr: LED power. 0 to 10.
* save: If True, the setting will persist across a power cycle.

### GY33_UART.query_raw()

Request for a single reading of the raw values.
When continuously output is turned off, you can use this to trigger a single read.

Note that you will still need to perform a "read" and a "get" to retrieve the values.

### GY33_UART.query_lcc()

Request for a single reading of the lcc values.
When continuously output is turned off, you can use this to trigger a single read.

Note that you will still need to perform a "read" and a "get" to retrieve the values.

### GY33_UART.query_processed()

Request for a single reading of the processed values.
When continuously output is turned off, you can use this to trigger a single read.

Note that you will still need to perform a "read" and a "get" to retrieve the values.

### GY33_UART.query_i2c()

Request for a single reading of the i2c address (...for use in I2C mode).
This value is never sent automatically, so you must send a query to obtain the i2c address.

Note that you will still need to perform a "read" and a "get" to retrieve the values.

### GY33_UART.set_baudrate(rate)

Sets the baudrate.

* rate: Must be either 9600 or 115200.

### GY33_UART.calibrate_white_balance()

Performs a white balance calibration.
The sensor should first be placed on a suitable white surface.

This is handled by the built-in micro-controller affects the processed RGB values.
The built-in microcontroller does not provide a black calibration.

### GY33_UART.set_i2c_addr(addr)

Sets the i2c address (...for use in I2C mode).

* addr: Desired i2c address. Must be between 0 to 127.

### GY33_UART.integration_time(time)

Sets the integration time in milliseconds.
A higher integration time will provide a higher resolution for the raw values, but at the expense of a lower update rate.

* time: Integration time in milliseconds. (Valid values are 700, 154, 100 (default), 24, or 2.4).


### GY33_UART.calibrate_white()

Performs white calibration.
The sensor should first be placed on a suitable white surface.

After calibration, the same surface should return 255 for all RGBC values when performing a "get_calibrated()".

### GY33_UART.calibrate_black()

Performs black calibration.
The sensor should first be placed on a suitable black surface.

After calibration, the same surface should return 0 for all RGBC values when performing a "get_calibrated()".

### GY33_UART.get_raw()

Returns a list containing the raw "Red, Green, Blue, Clear" values.

### GY33_UART.get_lcc()

Returns a list containing the "Lux (brightness), Color Temperature, Color" values.

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

### GY33_UART.get_processed()

Returns a list containing the processed "Red, Green, Blue" values.

### GY33_UART.get_i2c_addr()

Sets the i2c address as an integer (...for use in I2C mode).

## Examples

This first example uses continous output.

```python
import machine
import gy33_uart
import time

p0 = Pin(0, Pin.IN, Pin.PULL_UP)

uart = machine.UART(1, baudrate=9600, tx=4, rx=5)
gy33 = gy33_uart.GY33_UART(uart)

gy33.set_led(10) # Full power
gy33.set_output(True, False, True) # Continuously output raw and processed RGB values

while True:
    gy33.read()
    if p0.value() == 0:
        print(gy33.get_raw())
        print(gy33.get_processed())
        time.sleep(1)
```

The second example disables continuous output and sends a query when required.

```python
import machine
import gy33_uart
import time

p0 = Pin(0, Pin.IN, Pin.PULL_UP)

uart = machine.UART(1, baudrate=9600, tx=4, rx=5)
gy33 = gy33_uart.GY33_UART(uart)

gy33.set_led(10) # Full power
gy33.set_output(False, False, False) # Disables all continuously outputs

while True:
    if p0.value() == 0:
        gy33.query_raw()
        gy33.read(True)

        gy33.query_processed()
        gy33.read(True)

        print(gy33.get_raw())
        print(gy33.get_processed())
        time.sleep(1)
```
