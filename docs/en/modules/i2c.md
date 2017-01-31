# I²C Module
| Since  | Origin / Contributor  | Maintainer  | Source  |
| :----- | :-------------------- | :---------- | :------ |
| 2014-12-22 | [Zeroday](https://github.com/funshine) | [Zeroday](https://github.com/funshine) | [i2c.c](../../../app/modules/i2c.c)|

This module supports different interfaces for talking to I²C slaves. All interfaces can be assigned to arbitrary GPIOs for SCL and SDA and can be operated concurrently.
- `i2c.SW` software based bitbanging, synchronous operation
- `i2c.HW0` ESP32 hardware port 0, synchronous or asynchronous operation
- `i2c.HW1` ESP32 hardware port 1, synchronous or asynchronous operation

The hardware interfaces differ from the SW interface as the commands (start, stop, read, write) are queued up to an internal command list. Actual I²C communication is initiated afterwards using the `i2c.transfer()` function. Commands for the `i2c.SW` interface are immediately effective on the I²C bus and read data is also instantly available.

## i2c.address()
Send (`i2c.SW`) or queue (`i2c.HWx`) I²C address and read/write mode for the next transfer.

Communication stops when the slave answers with NACK to the address byte. This can be avoided with parameter `ack_check_en` on `false`.

#### Syntax
`i2c.address(id, device_addr, direction[, ack_check_en])`

#### Parameters
- `id` interface id
- `device_addr` device address
- `direction` `i2c.TRANSMITTER` for writing mode , `i2c.RECEIVER` for reading mode
- `ack_check_en` enable check for slave ACK with `true` (default), disable with `false`

#### Returns
`true` if ack received (always for ids `i2c.HW0` and `i2c.HW1`), `false` if no ack received (only possible for `i2c.SW`).

#### See also
[i2c.read()](#i2cread)

## i2c.read()
Read (`i2c.SW`) or queue (`i2c.HWx`) data for variable number of bytes.

#### Syntax
`i2c.read(id, len)`

#### Parameters
- `id` interface id
- `len` number of data bytes

#### Returns
- `string` of received data for interface `i2c.SW`
- `nil` for ids `i2c.HW0` and `i2c.HW1`

#### Example
```lua
id  = i2c.SW
sda = 1
scl = 2

-- initialize i2c, set pin1 as sda, set pin2 as scl
i2c.setup(id, sda, scl, i2c.SLOW)

-- user defined function: read from reg_addr content of dev_addr
function read_reg(dev_addr, reg_addr)
    i2c.start(id)
    i2c.address(id, dev_addr, i2c.TRANSMITTER)
    i2c.write(id, reg_addr)
    i2c.stop(id)
    i2c.start(id)
    i2c.address(id, dev_addr, i2c.RECEIVER)
    c = i2c.read(id, 1)
    i2c.stop(id)
    return c
end

-- get content of register 0xAA of device 0x77
reg = read_reg(0x77, 0xAA)
print(string.byte(reg))
```

####See also
[i2c.write()](#i2cwrite)

## i2c.setup()
Initialize the I²C module.

#### Syntax
`i2c.setup(id, pinSDA, pinSCL, speed)`

####Parameters
- `id` interface id
- `pinSDA` 0~33, IO index
- `pinSCL` 0-33, IO index
- `speed` bit rate in Hz, positive integer
  - `i2c.SLOW` for 100000 Hz, max for `i2c.SW`
  - `i2c.FAST` for 400000 Hz
  - `i2c.FASTPLUS` for 1000000 Hz

#### Returns
`speed` the selected speed

####See also
[i2c.read()](#i2cread)

## i2c.start()
Send (`i2c.SW`) or queue (`i2c.HWx`) an I²C start condition.

#### Syntax
`i2c.start(id)`

#### Parameters
`id` interface id

#### Returns
`nil`

####See also
[i2c.read()](#i2cread)

## i2c.stop()
Send (`i2c.SW`) or queue (`i2c.HWx`) an I²C stop condition.

#### Syntax
`i2c.stop(id)`

####Parameters
`id` interface id

#### Returns
`nil`

####See also
[i2c.read()](#i2cread)

## i2c.transfer()
Starts a transfer for the specified hardware module. Providing a callback function allows the transfer to be started asynchronously in the background and `i2c.transfer()` finishes immediately. Without a callback function, the transfer is executed synchronously and `i2c.transfer()` comes back when the transfer completed. Data from a read operation is returned from `i2c.transfer()` in this case.

First argument to the callback is the error code (0 = no error), followed by a string with data obtained from a read operation during the transfer or `nil`.

#### Syntax
`i2c.transfer(id[, cb_fn][, to_ms])`

#### Parameters
- `id` interface id, `i2c.SW` not allowed
- `cb_fn(err, data)` function to be called when transfer finished
- `to_ms` timeout for the transfer in ms, defaults to 0=infinite

#### Returns
- `string` of received data (or `nil` if no read) for synchronous operation
- `nil` for asynchronous operation

## i2c.write()
Write (`i2c.SW`) or queue (`i2c.HWx`) data to I²C bus. Data items can be multiple numbers, strings or lua tables.

Communication stops when the slave answers with NACK to a written byte. This can be avoided with parameter `ack_check_en` on `false`.

####Syntax
`i2c.write(id, data1[, data2[, ..., datan]][, ack_check_en])`

####Parameters
- `id` interface id
- `data` data can be numbers, string or lua table.
- `ack_check_en` enable check for slave ACK with `true` (default), disable with `false`

#### Returns
`number` number of bytes written

#### Example
```lua
i2c.write(0, "hello", "world")
```

#### See also
[i2c.read()](#i2cread)