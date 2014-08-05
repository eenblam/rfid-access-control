Access Terminal Firmware
========================

Firmware to run on the external nodes, typically featuring an RFID-RC522 reader
and potentially other peripheral such as a Keypad or an LCD screen. Some
may have an H-Bridge to switch or buzz a door strike.

They interface with a serial interface with a simple protocol with the host.

Based on Atmega8 (because I had a bunch lying around). Running on 3.3V as
this is the voltage needed by the RFID reader.

RFID-RC522
----------
The reader to interface is the RFID-RC522; there are a bunch available everywhere
and they are cheap - in the order of $5. They interface via SPI with the
microcontroller. Uses https://github.com/miguelbalboa/rfid to interface with
the RC522 board, but hacked to not depend on Arduino libraries.

It only uses a tiny subset of that library: to read the UID (which could
probably be simplified, because it is quite a chunk of code). But hey, it was
already there and I have 8k to waste.

LCD
---
If this terminal should have an LCD to display messages or interact wit the
user, then these are connected to PC0..PC5 to the microcontroller.
(TODO: make it #define-able which features are available)

![LCD connector][lcd]

The LCD typically has 14 or 16 connector pins.
   - **LCD 1** _(GND)_ to **GND**
   - **LCD 2** _(+5V)_ to **5V** (unfortunately, many displays only work with
      5V instead of 3.3V. Might need dual voltage on the board).
   - **LCD 3** _(contrast)_ to **GND**
       _the contrast is controllable with a resistor, but connecting it to GND
       is just fine_
   - **LCD 4** _(RS)_ to **PC4** (Pin 27 on PDIP Atmega8)
   - **LCD 5** _(R/-W)_ to *GND** _We only write to the display,
      so we set this pin to GND_
   - **LCD 6** _(Clock or Enable)_ to **PC5** (Pin 28 on PDIP Atmega8)
   - LCD 7, 8, 9, 10 are _not connected_
   - **LCD 11** _(Data 4)_ to **PC0** (Pin 23 on PDIP Atmega8)
   - **LCD 12** _(Data 5)_ to **PC1** (Pin 24 on PDIP Atmega8)
   - **LCD 13** _(Data 6)_ to **PC2** (Pin 25 on PDIP Atmega8)
   - **LCD 14** _(Data 7)_ to **PC3** (Pin 26 on PDIP Atmega8)
   - If availble: LCD 15 and LCD 16 are for background light.

Keypad
------
TBD
Sends `K` followed by keypress followed by <CR><LF>

Output Pins
-----------
Output bits in PC0..PC5 if there is no LCD display. (TBD: and some leftover pins)

Serial Protocol
---------------
Line based protocol. Right now, we end all lines with `<CR><LF>` because that
works right out of the box with any terminal emulator (e.g. minicom), without
too much configuration. We only use RX and TX, so hardware flow control needs
to be switched off.

The serial protocol communicates with 9600 8N1 (TODO: reconsider speed if we go
with RS232 instad of RS422 on the physical wire).

### Terminal -> Host

#### RFID

Whenever a new RFID card is seen, is sent to the host:

     Ixx yyyyyyyy<CR><LF>

(Example: `I07 c41abefa24238d`)
With xx being the number of bytes (RFID IDs come in 4, 7 and 10 bytes. The
number is written in hex, so values right now `04`, `07`, `0a`) followed
by the actual bytes. All numbers (xx and yy...) are in hex.

While the card is seen, this line is repeated every couple of 100ms

#### Keypad
(TBD)

Each Key pressed on the phonepad is transmitted with a single line

     K*<CR><LF>

The star representing the key in this case.

### Host -> Terminal

The terminal also responds to one-line commands from the host.
They are one-letter commands, followed by parameters and end
with a `<CR>` or `<LF>` or both.

     ?     : Prints help.
     W<xx> : Writes output pins. Understands 8-bit hex number, but only
             6 bits are currently addressed: PC[0..5] on the Atmega8
             Responds with W<yy> with <yy> the actual bits being set (some
             might not be available).
             Can be used to switch on fancy LEDs or even remotely trigger a
             relay or transistor to open the strike.
             Example:

             W ff

     M<r><msg> : Write a message to the LCD screen. Example:

                 M1Hello World

                 writes this message to the second line.
     r     : Reset RFID reader (Should typically not be necessary except after
             physical connection reconnect of its SPI bus).
     P     : Ping; responds with "Pong". Useful to check aliveness.

Responses generally are prefixed with the letter of the command. Makes
interfacing simple.

To compile, you need the avr toolchain installed:

     sudo aptitude install gcc-avr avr-libc avrdude

Work in progress :)

![Work in progress][work]

[work]: https://github.com/hzeller/rfid-access-control/raw/master/img/work-in-progress.jpg
[lcd]: https://github.com/hzeller/rfid-access-control/raw/master/img/lcd-connector.jpg