# Micropython GY-33 Driver

The GY-33 module uses a TCS3472 sensor, but does not expose the TCS3472 by default.
Instead, it contains a STM32F030F4 micro-controller which provides UART and I2C interface for the TCS3472 and the onboard LED.

A driver written for the TCS3472 will not normally work with this module, as the onboard micro-controller uses a different I2C address and registers.

I couldn't find any english documentation for the GY-33, so the below info are extracted from the chinese documentation found here https://pan.baidu.com/s/1kVq51dl

# Pinout

| Pin | Label | Description |
| --- | --- | --- |
| 1 | VCC | Power supply. 3v to 5v |
| 2 | CT | UART TX or I2C SCL |
| 3 | DR | UART RX or I2C SDA |
| 4 | GND | Ground |
| 5 | NC | Reserved, do not connect |
| 6 | INT | Interrupt for the TCS3472 (S1 must be grounded) |
| 7 | SDA | SDA for the TCS3472 (S1 must be grounded) |
| 8 | SCL | SCL for the TCS3472 (S1 must be grounded) |
| A | SO | Selection for UART / I2C mode |
| B | S1 | Disable MCU, enable TCS3472 when grounded |

## Pin S0

When grounded, the micro-controller will be I2C mode via pins 2 and 3 (CT and DR).
When unconnected or high, the micro-controller will be in UART mode via pins 2 and 3 (CT and DR).

## Pin S1

When grounded, the built-in micro-controller will be disabled, and you will be able to access the TCS3472 via pins 7 and 8 (SDA and SCL).
When unconnected or high, the TCS3472 will not be directly accessible, and pins 7 and 8 (SDA and SCL) should not be connected.

## UART and I2C mode

See their respective folders for more details.