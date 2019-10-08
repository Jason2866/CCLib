This fork from CCLib is meant to simplify the flashing of CC2530 devices with a Wemos D1 Mini or any equivalent based on ESP8266, to be used with [Tasmota](https://github.com/arendst/Sonoff-Tasmota).

It is based on this [fork](https://github.com/kirovilya/CCLib) from `kirovilya` and instructions described in this [blog post](https://www.zigbee2mqtt.io/information/alternative_flashing_methods.html).

The port speed is reduced to 19200 to avoid communication errors :`ERROR: Could not read from the serial port!`

Default pinout is the following:

- `GPIO4` connected to `CC_DC` aka `P2_2` or `Debug Clock`
- `GPIO5` connected to `CC_RST`
- `GPIO12`connected to `CC_DD` aka `P2_1` or `Debug Data`

[![layout](https://github.com/Jason2866/CCLib/blob/master/cc2530_pin_layout.png)](https://github.com/Jason2866/CCLib/blob/master)

[![Pinlayout](https://github.com/Jason2866/CCLib/blob/master/webee_cc2530_cc2591_pinlayout.png)](https://github.com/Jason2866/CCLib/blob/master)

Tasmota Zigbee currenlty only supports **ZNP 1.2 default mode** for coordinator. CCLib requires to remove the second last line.

For convenience, ready to use firmwares are provided, just unzip them and select the right one: `CC2530`, `CC2530 + CC2591` or `CC2530 + CC2592`.

Make sure you have Python 2.7 installed and install pyserial 3.0.1: `pip install pyserial==3.0.1`

Before flashing, check the connectivity:

```python Python/cc_info.py -p <serial_port>```

Example or result:

```
INFO: Found a CC2530 chip on /dev/cu.usbserial-xxxx

Chip information:
      Chip ID : 0xa524
   Flash size : 16 Kb
    Page size : 2 Kb
    SRAM size : 1 Kb
          USB : No

Device information:
 IEEE Address : 000000000000
           PC : 0000

Debug status:
 [ ] CHIP_ERASE_BUSY
 [ ] PCON_IDLE
 [X] CPU_HALTED
 [ ] PM_ACTIVE
 [ ] HALT_STATUS
 [X] DEBUG_LOCKED
 [X] OSCILLATOR_STABLE
 [ ] STACK_OVERFLOW

Debug config:
 [ ] SOFT_POWER_MODE
 [ ] TIMERS_OFF
 [ ] DMA_PAUSE
 [ ] TIMER_SUSPEND
```

To flash the firmware, use the following command: (flashing takes \~20 minutes)

```python Python/cc_write_flash.py -e -p /dev/cu.usbserial-1420 -i Bin/CC2530_DEFAULT_20190608_CC2530ZNP-Prod.hex```

```
INFO: Found a CC2530 chip on /dev/cu.usbserial-xxxx

Chip information:
      Chip ID : 0xa524
   Flash size : 256 Kb
    Page size : 2 Kb
    SRAM size : 8 Kb
          USB : No
Sections in Bin/CC2530_DEFAULT_20190608_CC2530ZNP-Prod.hex:

 Addr.    Size
-------- -------------
 0x0000   8176 B
 0x1ff6   10 B
 0x3fff0   1 B
 0x2000   239616 B

This is going to ERASE and REPROGRAM the chip. Are you sure? <y/N>:  y

Flashing:
 - Chip erase...
 - Flashing 4 memory blocks...
 -> 0x0000 : 8176 bytes
    Progress 100%... OK
 -> 0x1ff6 : 10 bytes
    Progress 100%... OK
 -> 0x3fff0 : 1 bytes
    Progress 100%... OK
 -> 0x2000 : 239616 bytes
    Progress 100%... OK

Completed
```

<BR>

Once flashed, the recommended connection to Wemos D1 mini is:

- `GPIO13` (Zigbee RX) connected to `CC_TXD` aka `P0_3`
- `GPIO15` (Zigbee TX) connected to `CC_RXD` aka `P0_2`

<BR>

---------

# CCLib

A set of utilities to convert your Arduino board to a CC.Debugger for flashing Texas Instruments' CCxxxx chips.
It currently supports the CC2530/40/41 chips ([compatibility table](#compatibility-table)) but with [your help it can support any chip](#contributing-other-chip-drivers) compatible with the CC.Debugger protocol.

Keep in mind but this more than just a set of utilities! It comes with complete, reusable Arduino and Python libraries for adding CC.Debugger support to your projects!



### Prepare your software

1. You will need Python 2.7 or later installed to your system
2. Open a terminal and change directory into the `Python` folder of this project
3. Install required python modules by typing: `pip install -r requirements.txt`
4. Test your set-up:
```
~$ ./cc_info.py -p [serial port]
```

If you see something like this, you are ready:

```
Chip information:
      Chip ID : 0x4113
   Flash size : 16 Kb
    SRAM size : 1 Kb
          USB : No

Device information:
 IEEE Address : 13fe41b61cde
           PC : 002f
```

However, if you see something like this, something is wrong and you should probably check your wiring and/or reset the arduino board or the CC board.

```
Chip information:
      Chip ID : 0x4113
   Flash size : 16 Kb
    SRAM size : 1 Kb
          USB : No

Device information:
 IEEE Address : 000000000000
           PC : 0000
```

### Using the software

The python utilities provide a straightforward interface for reading/writing to your CCxxxx chip:

* __cc_info.py__ : Read generic information from your CCxxxx chip. Usage exampe:
```
~$ ./cc_info.py -p /dev/ttyS0
```

* __cc_read_flash.py__ : Read the flash memory and write it to a hex/bin file. Usage example:
```
~$ ./cc_read_flash.py -p /dev/ttyS0 --out=output.hex
```

* __cc_write_flash.py__ : Write a hex/bin file to the flash memory. You can optionally specify the `--erase` parameter to firt perform a full chip-erase. Usage example:
```
~$ ./cc_write_flash.py -p /dev/ttyS0 --in=output.hex --erase
```

* __cc_resume.py__ : Exit from debug mode and resume chip operations. Usage example:
```
~$ ./cc_resume.py -p /dev/ttyS0
```

_NOTE:_ If you don't want to use the `--port` parameter with every command you can define the `CC_SERIAL` environment variable, pointing to the serial port you are using:

```
~$ export CC_SERIAL=/dev/ttyS0
```

## Compatibility Table

In order to flash a CCxxxx chip there is a need to invoke CPU instructions, which makes the process cpu-dependant. This means that this code cannot be reused off-the-shelf for other CCxxxx chips. The following table lists the chips reported to work (or could work) with this library:

<table>
    <tr>
        <th>Chip</th>
        <th>Chip ID</th>
        <th>Driver</th>
        <th>Status</th>
    </tr>
    <tr>
        <td>CC2530</td>
        <td><strong>0xa5</strong>..</td>
        <td>CC254X</td>
        <td>:white_check_mark: Works</td>
    </tr>
    <tr>
        <td>CC2531</td>
        <td><strong>0xb5</strong>..</td>
        <td>CC254X</td>
        <td>:large_orange_diamond: Looking for testers</td>
    </tr>
    <tr>
        <td>CC2533</td>
        <td><strong>0x95</strong>..</td>
        <td>CC254X</td>
        <td>:large_orange_diamond: Looking for testers</td>
    </tr>
    <tr>
        <td>CC2540</td>
        <td><strong>0x8d</strong>..</td>
        <td>CC254X</td>
        <td>:white_check_mark: Works</td>
    </tr>
    <tr>
        <td>CC2541</td>
        <td><strong>0x41</strong>..</td>
        <td>CC254X</td>
        <td>:large_orange_diamond: Looking for testers</td>
    </tr>
    <tr>
        <td>CS2510</td>
        <td><strong>0x81</strong>..</td>
        <td>CS2510</td>
        <td>:large_orange_diamond: Looking for testers</td>
    </tr>
</table>

### Contributing other chip drivers

Since the arduino sketch is quite simple, it's possible to support any CCxxxx device solely by creating a new chip driver. Even if your chip uses a different debug protocol instruction set (such as CC2510) you can modify it on-the-fly.

In order to create a new chip driver you should create a new file in the `Python/cclib/chip` folder with the name of your chip (for example `cc2510.py`), and create a new Python class, subclassing from the `ChipDriver` class. For example:

```python
class CC2510(ChipDriver):
    """
    Chip-specific code for CC2510 SOC
    """

    @staticmethod
    def test(chipID):
        """
        Check if this ChipID can be handled by this class
        """
        return ((self.chipID & 0xff00) == 0x8100)

    def chipName(self):
        """
        Return Chip Name
        """
        return "CC2510"

    def initialize(self):
        """
        Initialize chip driver
        """

        # Get chip info
        self.chipInfo = self.getChipInfo()

        # Populate variables
        self.flashSize = self.chipInfo['flash'] * 1024
        self.flashPageSize = 0x400
        self.sramSize = self.chipInfo['sram'] * 1024
        self.bulkBlockSize = 0x800
        self.flashWordSize = 2

```

And you must then register your class in the `Python/cclib/ccdebugger.py`

```python
# Chip drivers the CCDebugger will test for
from cclib.chip.cc2540x import CC254X
from cclib.chip.cc2510 import CC2510
CHIP_DRIVERS = [ CC254X, CC2510 ]
```

After that you need to implement all the functions exposed by the `ChipDriver` (available in `Python/cclib/chip/__init__.py`), but you can just copy the `cc2540x.py` driver and work on top of it.

We are looking forward for your support for new chips!

## Protocol

The protocol used between your computer and your Arduino is quite simple and not really fault-proof. This was intended as a pure proxy mechanism in order to experiment with the CC Debugging protocol from the computer. Therefore, if you interrupt any operation in the middle, you will most probably have to unplug and re-plug your Teensy/Arduino. That said, here is the protocol:

Since most of the debug commands are at max 4-bytes long, we are sending from the computer a constant-sized frame of 4-bytes:

    +-----------+-----------+-----------+-----------+
    |  Command  |   Data 0  |   Data 1  |   Data 2  |
    +-----------+-----------+-----------+-----------+

The only exceptions are:

  * The brust-write command (CMD_BRUSTWR), where up to 2048 bytes might follow the 4-byte frame, and
  * The instrunctionset update command (CMD_INSTR_UPD), were 16 bytes must follow the 4-byte frame.

The Teensy/Arduino will always reply with the following 3-byte long frame:

    +-----------+-----------+-----------+
    |   Status  |    ResH   |  Err/ResL |
    +-----------+-----------+-----------+

If the status code is `ANS_OK(1)`, the `ResH:ResL` word contains the resulting word (or byte) of the command. If it's `ANS_ERR(2)`, the `ResL` byte contains the error code.


## Disclaimer

Users have successfully flashed various BlueGiga BLE112/BLE113 (CC2540) modules with this solution, however the developers DO NOT GUARANTEE THAT THIS WILL WORK IN YOUR CASE! **The developers cannot be held liable for any damage caused by using this library, directly or indirectly. YOU ARE USING THIS CODE SOLELY AT YOUR OWN RISK!**

## License

Copyright (c) 2014-2016 Ioannis Charalampidis

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
