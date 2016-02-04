# The W65C265SXB Memory System

The CPU at the heart of the 265SXB, the 65816, provides 24 address lines and can
address up to 16 MB of memory. Internally, this space is segemented into 256
"banks" of 64K each. When the CPU is emulating the 6502, only the first bank
(first two hex digits are "00", also known as _Bank Zero),_ is accessed. See the
65816 Programming Manual for more details.

If this sounds confusing, don't worry, the memory settings of the 265SXB are
even worse. The easist way to understand them is to start with the "out of the
box" configuration and then to work our way through the various settings to
increase the address space. 

## Out of the box

The 265SXB is shipped with the following memory configuration:

### RAM

The W65C265S microcomputer comes with 576 bytes of RAM on-chip, from 00:0000 to
00:01FF (512 bytes) and from 00:DF80 to 00:DFBF (64 bytes). The 265SXB also
includes 32K of SRAM as an AS7C256A-10TIN (U2 on the upper left) enabled by
default that span from 00:0000 to 00:7FFF.

> Note: Technically, there is a difference if the 265SXB uses on-chip RAM for
> 00:0000 to 00:01FF or the SRAM, but this is irrelevant for us at the moment.

You can test if the 32K on-board SRAM is working by using the Monitor to modify
the memory at, say, 00:7000 and then reading it back. 

### ROM 

The 265SXB comes with 8K of burned-in on-chip ROM that holds the [Mensch
Monitor](https://github.com/scotws/265SXB-Guide/blob/master/monitor.md), a
simple operating system. This is located at 00:E000 to 00:FFFF and includes the
various system vectors for interrupts. 


### I/O and peripheral registers

All non-CPU registers are found in the area from 00:DF00 to 00:DFFF together
with 64 bytes of RAM as described above. 


## Adding More Memory

### Adding Flash ROM

The 265SXB comes with an unpopulated PLCC socket designed to hold a
SST39SF010A 128K x 8 flash memory chip (70nSec or faster; U3 in upper right
corner of the board). When installed, 32K from the Flash ROM is mapped into the
address space between 00:8000 and 00:FFFF. How much is visible depends on
whether the mask ROM (00:E000 - 00:FFFF) and peripheral registers/on-chip RAM
(00:DF00 - 00:DFFF) are enabled.

### Selecting a 32K bank of Flash ROM

Which 32K bank of the 128K Flash ROM is accessible depends on the settings
of the FAMS (PD4<4>) and FA15 (PD4<3>) signals. On reset the pins of Port 4 are
all configured as inputs (Port 4 Data Direction Register PDD4 00:DF24 contains
$00) which allows external resistors to pull the signals high.

To select another bank, the PDD4<4:3> bits must be changed to make one or
both of the pins outputs and the corresponding bit or bits in PD4 (00:DF20)
should be set to zero to make the signal low.

### Switching off the built-in 8K ROM 

On reset the BCR register (00:DF40) is set to $01 which enables the internal
ROM. To disable it bit 7 of the BCR register must be set. For example: 
```
LDA #$80     ; Disable Mensch ROM
TSB BCR
```
The internal ROM uses interrupts so user code must disable interrupt
generation, execute an SEI to ignore interrupts, or provide its own interrupt
vectors until the internal ROM is renabled.
```
LDA #$80     ; Enable Mensch ROM
TRB BCR
```
![Memory Map]
(https://github.com/scotws/265SXB-Guide/blob/master/Images/W65C256SXB_Memory_2.png)

_Image: Bank 00 showing the three simplest memory configurations. Light: RAM,
dark: ROM, red: I/O registers and 64 byte on-chip RAM, grey: empty._

### How the Chip Select Signals are Generated

Chip selection on the 265SXB is handled by lines that are attached to Port 7. In
contrast to the other ports, it is output only. The Port 7 Chip Select Register
(PCS7 at 00:DF27) determines how the individual lines of this port are used: A
"0" bit (not set) turns that line into a normal output port of the associated
data register (PD7 at 00:DF23). For example, if PCS7, bit 3 is "0", whatever is
in PD7, bit 3 will be on the pin. However, a "1" bit (set) in PCS7 will enable
that pin as a chip select line. 

The Data Sheet for the 265SXP gives a detailed table for the individual lines.
Some of these are used internally. The ones that are interesting for us are:

- **CS5B** selects the 4M address space in the banks $00 to $3F (pin 75). Note there
are various "holes" punched in this space for the Flash ROM and other stuff at
Bank 0. 

- **CS6B** selects the 8M address space in the banks $40 to $BF (pin 76).

- **CS7B** selects the 4M address space in the banks $C0 to $FF (pin 77)

These three signals are also exposed through the 50-pin XBus265 commector
(connector J1). 

On hardware reset, the system first sets PCS7 to $00 and PD7 to $FF. However,
during boot, the Mensch Monitor changes PCS7 to $FB and PD7 to $04 (see ROM
listing at 00:E14B). In this configuration, bit 2 is kept as "1", which makes
sure the system uses the on-chip interrupt vectors, on-chip ROM, on-chip RAM,
timer, control and other regiesters from 00:DF00 to 00:DFBF and 00:0000 to
00:01FF.
