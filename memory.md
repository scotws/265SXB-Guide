# The W65C265SXB Memory System

The CPU at the heart of the 265SXB, the 65816, provides 24 address lines and can
address up to 16 MB of memory. Internally, this space is segemented into 256
"banks" of 64k each. When the CPU is emulating the 6502, only the first bank
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
includes 32k of SRAM as an AS7C256A-10TIN (U2 on the upper left) enabled by
default that span from 00:0000 to 00:7fff.

> Note: Technically, there is a difference if the 265SXB uses on-chip RAM for
> 00:0000 to 00:01FF or the SRAM, but this is irrelevant for us at the moment.

You can test if the 32k on-board SRAM is working by using the Monitor to modify
the memory at, say, 00:7000 and then reading it back. 


### ROM 

The 265SXB comes with 8k of burned-in on-chip ROM that holds the [Mensch
Monitor](https://github.com/scotws/265SXB-Guide/blob/master/monitor.md), a
simple operating system. This is located at 00:E000 to 00:FFFF and includes the
various system vectors for interrupts. 


### I/O and peripheral registers

All non-CPU registers are found in the area from 00:DF00 to 00:DFFF together
with 64 bytes of RAM as described above. 

![Memory Map]
(https://github.com/scotws/265SXB-Guide/blob/master/Images/W65C256SXB_Memory.png)

_Image: The two simplest memory configurations. Light: RAM, dark: ROM,
red: I/O registers and 64 byte on-chip RAM; grey: empty._


## Enabling further address space

The 265SXB uses internal registers to enable access to further memory. The most
important of these is the Chip Select enable register (CS) at 00:DF27 with an
inital value of $FB. A **clear** (0) bit selects:

- **CS0B** External Port replacement (32 byte). Inactive when BCR0 = 0 (see
  below) and 00:DF00 to 00:DF07 are used for internal I/O register selection
  (P70). When BCR0 = 1 the external memory bus is enabled and CS0B is active for
  the addresses 00:DF00 to DF1F. _Usually leave this unchanged at bit set (1)._

- **CS1B** Select coprocessor expansion. Affects the 64 bytes from 00:DFC0 to
  00:DFFF (P71). _Leave unchanged at bit set (1) unless you really know what you
  are doing._

- **CS2B** Select on-chip interrupt vectors, on-chip ROM, on-chip RAM, timer,
  control and other regiesters from 00:DF00 to 00:DFBF and 00:0000 to 00:01FF
  (P72).  Includes the Mensch Monitor ROM at 00:E000, see CS4B below. _Leave
  unchanged at bit clear (0) unless you want to replace the Monitor, see below
  for discussion._

- **CS3B** Select "cache" memory 00:0200 to 00:7FFF. You might think this is
  enabled (bit clear, 0) on the 265SXB, but in fact it is _disabled_ (bit set,
  1). In fact, clearing this bit is _disable_ the SRAM. In neither case does the
  LED go on, even though it should based on the [board
  schematic](http://www.westerndesigncenter.com/wdc/Schematics/W65C265SXB.pdf).
  This is Pin 73 of the microcomputer. _Leave it the way it is to use the
  on-board 32k SRAM._

- **CS4B** Select ROM from 00:8000 to 00:FFFF, which is the Flash ROM socket
  (see below, P74). Default is disabled (bit set, 0). Note that if we selected
  the on-chip burned-in ROM with CS2B (the default setting), only 24k of the
  Flash ROM from 00:8000 to 00:DEFF can be selected. The on-chip addresses
  00:DF00 to 00:DFFF will _never_ appear in this selection. _Select (clear bit,
  0) to enable Flash ROM, see below._

- **CS5B** Select the 4M address space in the banks 00 to 3F (P75). If CS2B,
  CS3B, and/or CS4B are selected, these will "punch holes" in this address
  range, that is, they are not selected. _Probably not something you will use,
  leave unselected (bit set, 1)._

- **CS6B** Select the 8M address space in the banks 40 to BF (P76). _Select by
  clearing this bit (change to 0) if you want to populate this address space._

- **CS7B** Select the 4M address space in the banks C0 to FF (P77). Note the
  Mensch Monitor code listing contains an error in the comments here with the
  claim that the banks selected are C0 to "CF". _Select by clearing this bit
  (change to 0) if you want to populate this address space._

We'll now examine these steps in more detail.


## Adding Flash ROM

The 265SXB comes with an unpopulated PLCC socket designed to hold a
SST39SF010A 128k x 8 flash memory chip (70nSec or faster; U3 in upper right
corner of the board). When installed, 32k from the Flash ROM is mapped into the
address space between 00:8000 and 00:FFFF. How much is visible depends on
whether the mask ROM (00:E000 - 00:FFFF) and peripheral registers/on-chip RAM
(00:DF00 - 00:DFFF) are enabled.

Which 32K bank of the 128K Flash ROM is accessible depends on the settings
of the FAMS (PD4<4>) and FA15 (PD4<3>) signals. On reset the pins of PORT4
are all configured as inputs (Port 4 Data Direction Register PDD4 00:DF24
contains $00) which allows external resistors to pull the signals high.

To select another bank the PDD4<4:3> bits must be changed to make one or
both of the pins outputs and the corresponding bit or bits in PD4 (00:DF20)
should be set to zero to make the signal low.


## Switching off the built-in 8k ROM 

On reset the BCR register (00:DF40)is set to $01 which enables the internal
ROM. To disable it bit 7 of the BCR register must be set. For example: 
```
LDA #$80     ; Disable Mensch ROM
TSB BCR
```
The internal ROM uses interrupts so user code must disable interrupt
generation or execute an SEI to ignore interrupts until the internal ROM is
renabled.
```
LDA #$80     ; Enable Mensch ROM
TRB BCR
```

(System Speed Control Register (SSCR, $DF41) contains External RAM Select as bit
2; default entry is $FB)
