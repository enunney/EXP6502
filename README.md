# EXP6502
6502 based expandable computer.

<p align="center">
  <img src="./png/EXP6502 3D view.png" width="250" />
</p>

## Specifications

- W65C02 CPU @ 8MHz
- 64 kB SRAM
- 32 kB EEPROM (read only)
- 7 general purpose expansion ports
- Bank switching with access to up to 160 kB of memory
- W65C22 Versatile Interface Adapter
- Software controllable glitchless clock switching (8 MHz / fraction of 8 MHz)
- Built-in beeper speaker and LED
- 5 built-in push buttons with interrupt capability

## Architecture
### Block diagram

<p align="center">
  <img src="./png/Block diagram.png" width="500" />
</p>

The CPU address and data buses are directly connected to the onboard memory and peripherals. Tristate buffers isolate the local buses from the expansion ports.
CMOS logic ICs decode the address bus as well as the status of the control register to provide control signals for each component.

### Memory map

<p align="center">
  <img src="./png/Memory map.png" width="500" />
</p>

The lower half of the address space contains 32 kB of RAM (the low RAM) and the I/O space. The I/O space sits
between $6000 and $6FFF. It can be switched out to access the 4 kB of RAM located at the same memory addresses.

The I/O space is made of two parts. The bottom one is divided in 8 pages, each of these controlling one I/O
peripheral. The first I/O page is reserved for the built in VIA. The 7 others are each attributed to a different
expansion port. The top part of the I/O space, IOH, is available on every expansion port, in case they need more I/O
addresses.

The upper half of the address space is divided in two blocks, each of them is multiplied in 4 banks. One bank in each
block can be selected at a time, and the two blocks can have a different bank selected.

The first bank gives access to the ROM of the computer, it is selected by default at start up or after reset.
The second one gives access to 32 kB of RAM, the high RAM.
The third and fourth ones give access to memory placed on the expansion ports. The chip select signals of these
banks are accessible on any expansion port.


The bank switching in the lower half of the address space is controlled by the RIO signal, bit 3 of the control register.
It allows switching the window between $6000 and $6FFF to the I/O space, when RIO is low, or to the low RAM,
when RIO is high.

The bank switching in the upper half of the address space is controlled by the LB0, LB1, HB0 and HB1 signals, bits 4 to
7 of the control register. The bits LB0 and LB1 control the bank switching in the bottom block (\$8000 to \$BFFF). HB0
and HB1 control the switching in the top block (\$C000 to \$FFFF).
- The first bank, bank 0, is mapped to the ROM, in the two blocks.
- The bank 1, is mapped in the two blocks to the high RAM.
- The bank 2 enables the external high memory 0, EXT0HI in the top block and the external low memory 0, EXT0LO in the bottom block.
- The bank 3 enables the external high memory 1, EXT1HI in the top block and the external low memory 1, EXT1LO in the bottom block.

### Expansion port

The computer has 7 general purpose expansion ports.
They give access to buffered versions of the address and data buses of the computer, to various control signals of
the microprocessor and to the selecting signals for the external memory banks. They also have access to some pins
of the VIA.

The expansion ports have access to the main clock, φ2, as well as the oscillator clock and the divided clock. This
feature allows an expansion card to keep using one of these clocks while the computer has switched to another one,
for instance for slow peripherals that couldn’t handle the oscillator frequency. It is worth noting that the CPU can’t
access these peripherals while the clocks are different, as it would have an unpredictable behavior.

Any expansion port can take control of the computer, by setting low the EXTCTLB signal, which disables the busses of
the CPU and reverses the direction of the address and data buffers. The RDY signal should be put low at the same
time to effectively halt the CPU.

The expansion port 1 has an additional two pins header socket placed next to it. It is intended to be connected to the
read and write pins of a TFT LCD screen (controlled by an ILI9341 IC or compatible) for write only access when the
port 1 is connected to such a screen.

The pinout of the expansion ports and the function of each pin can be found in the technical reference manual.

### Control register

The control register allows the software to switch the memory banks, to change the clock speed and to directly
control an LED and the beeper speaker.
It is located at address $0000 of the address space, in the zero page of the 6502 microprocessor, to allow faster
access.
The function of each of its bits is given below:

- Bit 0: CPULED
→ Control signal for the built in LED.
- Bit 1: CPUSPK
→ Can be connected to the built in beeper speaker.
- Bit 2: SELCLK
→ Control the current clock source for the main clock φ2.
- Bit 3: RIO
→ Select the I/O space or the low RAM between $6000 and $6FFF.
- Bit 4: LB0
→ LSB of the number of the selected bank in the bottom block of the upper half of the address space.
- Bit 5: LB1
→ MSB of the number of the selected bank in the bottom block of the upper half of the address space.
- Bit 6: HB0
→ LSB of the number of the selected bank in the top block of the upper half of the address space.
- Bit 7: HB1
→ MSB of the number of the selected bank in the top block of the upper half of the address space.

More details are available in the technical reference manual.

### Clocking

The main clock of the computer, φ2, can be switched between two different sources: the oscillator clock or a divided
version of the oscillator clock.

The default oscillator frequency is 8 MHz. A counter and a JK flip-flop generate a divided version of this clock. The
dividing ratio can be configured with 4 jumpers on the computer. The ratio is between /2 when the value configured
by the jumpers is set to 15, and /32 when the value is 0.
The dividing ratio follows the formula: R = 2*(16-n), where n is the value configured with the jumpers.

The oscillator clock can also be replaced by an external clock, placed on any of the expansion port. The external clock
must be selected with the corresponding jumper.

### I/O

The computer has a built in Versatile Interface Adapter (VIA), of which the two GPIO ports are accessible on 2 pin
header sockets. The CA1, CA2, CB1 and CB2 pins of the VIA are accessible on another pin header socket, as well as on
every expansion port.

The bits 3, 4 and 5 of the PORT B of the VIA can be connected to an 8 to 3 priority encoder with jumpers. The inputs
of the encoder are driven by 5 push buttons. When a button is pressed, the corresponding code is sent to the VIA.
The encoder can generate an interrupt either directly or through the CA2 pin of the VIA, depending on the jumper
configuration.

The VIA can also drive the built in beeper speaker when the jumper connected to the speaker is set accordingly.
The control register is connected to the red LED and can be connected to the speaker.