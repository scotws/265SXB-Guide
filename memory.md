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

### RAM

The W65C265S microcomputer comes with 576 bytes of RAM in-chip, from 00:0000 to
00:01FF and 00:DF80 to 00:DFBF. The 265SXB includes 32k of SRAM as chip U2 (on
the upper left) enabled by default that span from 00:0000 to 00:7fff.

> Note: Technically, there is a difference if the 265SXB uses on-chip RAM for
> 00:0000 to 00:01FF, but this is irrelevant for us at the moment.

You can test if the 32k on-board SRAM is working by using the Monitor to modify
the memory at, say, 00:7000 and then reading it back. 

### ROM 

The 265SXB comes with 8k of burned-in ROM that holds the [Mensch
Monitor](https://github.com/scotws/265SXB-Guide/blob/master/monitor.md), a
simple operating system. This is located at 00:E000 to 00:FFFF and includes the
various system vectors for interrupts. 



## Adding Flash ROM

## Switching off the built-in 8k ROM 

(Bus Control Register (BCR, $DF40); default is $01, internal ROM enabled)
(System Speed Control Register (SSCR, $DF41) contains External RAM Select as bit
2; default entry is $FB)
