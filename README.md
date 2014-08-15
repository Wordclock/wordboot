# wordboot

This is a bootloader named *wordboot*, which is developed as part of the
[Wordclock][1] project. It is based upon the [optiboot][2] bootloader from the
[Arduino][3] project. By being compatible with [avrdude][4] it offers platform
independence. *wordboot* uses the frontpanel from the Wordclock to visually
indicate the current status.

## FEATURES

- [avrdude][3] compatibility: The bootloader implements the original STK500
  protocol as described in [AVR061: STK500 Communication Protocol][5] and hence
  is well supported by avrdude (and possibly other programming utilities).

- EEPROM support: In addition to read and write operations to the flash memory,
  it is also possible to read from and write to the EEPROM memory.

- Smallish: The complete bootloader fits into only **512** words of program
  memory, leaving enough space for the actual application.

- Visual indication: The current status (during boot-up and data transfer) is
  being signaled using the four available minute LEDs of the frontpanel.

- Configurable: Several options are available that influence the behaviour of
  the bootloader. They are described in more detail in the
  [appropriate](#configuration) section.

- Written in C: Unlike many other bootloaders for the AVR ATmega series,
  *wordboot* is written completely  in C, making it more easy to understand and
  maintain. It relies upon functionality provided by [avr-libc][6], upstreaming
  a great deal of complexity.

## REQUIREMENTS

The code itself is tested and developed with avr-gcc 4.9.x, avr-binutils 2.24.x
and avr-libc 1.8.x in mind. Other toolchains and/or versions might not work as
expected, especially since there are some restrains to the size the bootloader
is allowed to utilise.

This bootloader requires the `BOOTRST` fuse to be programmed. The boot flash
section size should be set to **512 words**, i.e. the boot start address is
`0x3e00` (word address, `0x7c00` as byte address). This results in the
following recommended fuse values for the Wordclock firmware:

- **lfuse**: `0xE2`
- **hfuse**: `0xDC`
- **efuse**: `0xFC`

## DOCUMENTATION

The source code is documented quite heavily using [Doxygen][7]. An appropriate
Doxyfile is provided along with the sources and can be found within the `doc/`
directory. It can be used to generate a HTML and/or PDF reference.

## CONFIGURATION

There are several options directly influencing the behaviour of the bootloader:

- **BAUD_RATE**: The default value is *9600*. It was chosen as the actual
  Wordclock firmware uses the same baud rate, so no change is needed in case a
  Bluetooth module is used. Baud rates up to *115200* have been tested
  successfully. Effectively this is only limited by the hardware module. Keep
  in mind, though, that the Wordclock project uses the internal RC oscillator,
  which is not ideal for UART transmissions.

- **LED_START_FLASHES**: The default value is *3*. This defines the number of
  LED flashes when the bootloader is entered and is meant to be an assistance
  for deciding when to issue the flash command using [avrdude][4].

- **LED_DATA_FLASH**: The default value is *1* (enabled). This defines whether
  the LEDs should flash when data is being received. This is an indicator that
  everything is working as intended. To disable this feature, remove the
  define completely.

- **ENABLE_RGB_SUPPORT**: The default value is *1* (enabled). This defines
  whether the bootloader should be built with RGB support, i.e. the minute
  LEDs will appear in white as all channels (red, green, blue) will be enabled
  simultaneously. Otherwise only the red channel will be used, which might be
  confusing as the Wordclock firmware itself is also flashing the minute LEDs
  during bootup.

- **BOOTLOADER_TIMEOUT_MS**: The default value is *1000* (1 second). This
  defines the time after which the bootloader is exited and the actual firmware
  itself is started. The timeout is reset with each received character, so the
  application will effectively be started when nothing is received for this
  period of time.

The options are expected to be changed within the Makefile (as CFLAGS),
although it is also possible to change them directly within the source by
defining appropriate macros.

## BUILDING

Building the bootloader requires the appropriate sections to be located
correctly. A suitable Makefile is provided taking care of this, while also
making sure that the result stays small by disabling unneeded options
enabled by default.

To build the firmware simply run `make` for the `wordclock` target within the
`src` directory:

    cd src/
    make wordclock

This will generate a `wordboot.hex` file, which can be flashed to the
microcontroller in the conventional way (i.e. ISP/HVPP). The `wordboot.lst`
file contains the resulting assembler listing and can be useful for debugging
purposes.

## USAGE

If the bootloader has been flashed successfully to the MCU and the fuses are
programmed correctly, the four minute LEDs should blink `LED_START_FLASHES`
times whenever the bootloader is entered (i.e. when powering up the Wordclock
or when restarting it using the appropriate UART command). The bootloader is
now accessible for as long as defined in `BOOTLOADER_TIMEOUT_MS`.

While the bootloader is accessible, an application can be flashed using the
following command:

    avrdude -p m328p -c arduino -b 9600 -P /dev/ttyUSB1 -U flash:w:Wordclock.hex

Make sure that the baud rate (-b) matches the configured setting. After a
successful flash the microcontroller performs a reset cycle and should start
the actual application briefly afterwards.

## UTILITIES

The provided `reset_wordboot.sh` script within the `util` directory can be used
to restart the Wordclock programmatically. It sends the appropriate UART
command to the given device. If no device was specified `/dev/ttyUSB0` is
chosen. The baud rate can be specified as a second parameter and defaults to
9600.

    ./reset_wordclock.sh /dev/ttyUSB1 9600

## CONTRIBUTIONS

The source code itself is maintained using git. The project along with its
[repositories][8] lives over at github.com. Contributions of any kind are
highly welcome. The simplest and fastest way for these kind of things is to use
the means provided by github.com itself, which allows for reporting bugs and/or
requesting features. It is also the preferred way to submit patches in form of
pull requests. If you are new to git and/or are not familiar with this process,
refer to [this][9] for a detailed description on how to submit a pull request.

In case you are looking for something to work on, you probably want to take a
look at the `TODO` file within the projects root directory. More recently the
github.com [issue tracker][10] is used to keep track of issues and coordinate
the development of new features. There is enough work left to do, so feel free
to start hacking away ;).

There is also a [repository][11] over at gitorious.org, which currently is
being used only as a mirror.

## DONATIONS

[![Flattr this git repo](http://api.flattr.com/button/flattr-badge-large.png "Flattr This!")](https://flattr.com/submit/auto?user_id=johnpatcher&url=https://github.com/Wordclock/wordboot)

[![PayPal donation](https://www.paypalobjects.com/en_US/i/btn/btn_donate_SM.gif "PayPal")](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=karol%40babioch%2ede&lc=DE&item_name=Wordclock&no_note=0&currency_code=EUR&bn=PP%2dDonationsBF%3abtn_donateCC_LG%2egif%3aNonHostedGuest)

Bitcoin: `1Dn6MEKgRAgdRS8Aeg88fug9XmdgTRpCDA`

## LICENSE

[![GNU GPLv3](http://www.gnu.org/graphics/gplv3-127x51.png "GNU GPLv3")](http://www.gnu.org/licenses/gpl.html)

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

[1]: https://github.com/Wordclock/
[2]: https://code.google.com/p/optiboot/
[3]: http://www.arduino.cc/
[4]: http://www.nongnu.org/avrdude/
[5]: http://www.atmel.com/Images/doc2525.pdf
[6]: http://www.nongnu.org/avr-libc/
[7]: https://www.stack.nl/~dimitri/doxygen/
[8]: https://github.com/Wordclock
[9]: https://help.github.com/articles/using-pull-requests
[10]: https://github.com/Wordclock/wordboot/issues
[11]: https://gitorious.org/wordclock/wordboot

