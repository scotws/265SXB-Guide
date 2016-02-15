# The W65C265SXB's built-in monitor program

The 265SXB is shipped with a small built-in operating system, the Mensch Monitor
(MM). As of time of writing (Jan 2016), the build shipped with the board was
2.0 from 1995. Yes, that made it more than 20 years old.

The MM is "burned" into the board and cannot be removed, replaced, or
overwritten. 

> Because of this, no backup of the MM is needed. 

However, you are able to jump from it to other ROM areas (if installed) and even
mask it completely; see the chapter [on
memory](https://github.com/scotws/265SXB-Guide/blob/master/memory.md) for
details.


## The Manual

The [Mensch Monitor
Manual](http://www.westerndesigncenter.com/Wdc/documentation/265monrom.pdf),
formally the *W65C265S Monitor ROM REFERENCE MANUAL* describes accessing and
using the Monitor. At time of writing this document, the current version was
Release 2.0 from February 10, 1995. 

> When working with the Manual, you should be aware that it is incomplete in
> places and misleading in others. For example, it states that the PUT_CHR 
> routine will print the character to Port 3 -- in fact, it will happily 
> print the character out through the USB connection as we saw in the 
> first chapter in the example when we printed an "a" to the screen. You shold
> therefore get used to consulting the source code listing.


## The Code Listing

WDC also provides a [full
listing](http://www.westerndesigncenter.com/wdc/documentation/265iromlist.pdf) Assembler
code listing of the Mensch Monitor) of the Mensch Monitor which you will be
referencing a lot. Note at the end there is a symbol table you can use to find
where jump routines are located. 


## Accessing the Built-In Routines

The Mensch Monitor provides a whole number of subroutines that can be accessed
from user programs. Note that most of these must be accessed through a long
subroutine jump - for example, ```JSL 00E04B``` to print a character. They
assume that the processor is in Native Mode, not Emulated as a 6502.

### Useful locations and variables

**SIN_BUF2** Base location of input buffer for various input routines, initially
set at 00:014D. Relevant code starts at 00:E2CF. 

**SINCNT2** Size of input buffer for various routines, initially set at 10
characters (decimal). Relevant code starts at 00:E2BC.

**SINEND2** Index for the last character removed from buffer.

**SININDX2** Index for last character placed in buffer. 


### Especially useful routines

**GET_BYTE_FROM_PC** ("get byte from input buffer", 00:E033, main code at
00:FBF9) - Returns a character from the input buffer. This routine must be
called with a long subroutine jump as ```JSL 00E033```. If there is data in the
buffer, it is returned with the carry bit clear and the data in the 8-bit
accumulator. Otherwise, carry bit is set, and 8-bit accumulator contains 00
(no data) or 80 (CONTROL-C or ESC received).

**GET_CHR** ("get character", 00:E036, main code at 00:F1F9) - Get a single
character from the console. Note this routine must be called with a long
subroutine jump as ```JSL 00E036```. Returns with the carry bit clear and the
character received in 8-bit A register.

**GET_PUT_CHR** ("get a character and echo to console", 00:E03C, main code at
00:F1C9) Gets a character from the input buffer and echos it back. This routine
must be called with a long subroutine jump as ```JSL 00E03C```. Returns with the
received character in 8-bit A register and the carry bit clear. Calls
GET_BYTE_FROM_PC. 

**GET_STR** ("get string", 00:E03F, main code at 00:F3D7) - Receives a string
from the console until either ENTER or ESC is received and stores the string
with a terminating 00 byte to a specified buffer. This routine must be called
with a long subroutine jump as ```JSL 00E03F```. The address of the buffer must
be provided by the bank byte in the 8-bit A register and the rest of the address
in 16-bit X register. No arguments are returned. If the string entry was ended
by an ENTER, the carry bit will be clear. If it was ended by an ESC, the carry
bit will be set. Calls GET_PUT_CHR to echo the string.

**PUT_CHR** ("put character", 00:E04B, main code at 00:FDBC) - Output a single
character to the console. Note that this routine must be called with a long
subroutine jump as ```JSL 00E04B```. See [the example
at](https://github.com/scotws/265SXB-Guide/blob/master/simple_programs.md)
Simple Programs. Though the Manual states that the character is printed to port
3, it will actually work fine with the basic USB port. 

**PUT_STR** ("put string", 00:E04E; main code at 00:F3A1) - Output a string to
the console. Must be called with long subroutine jump as ```JSL 00E04E```. Put
the bank of the string address in the 8-bit A register, and the rest of the
address in the 16-bit X register. The string must be terminated by a zero byte
(00, ASCII "null"). This routine calls PUT_CHR.  
```
  ; PUT_STR example program
        .origin 2000            ; start at 00:2000
        .mpu 65816              ; tells assembler this is a 65816 MPU

        clc                     ; switch to native mode
        xce 
        sep 20                  ; switch A register to 8 bit
        rep 10                  ; switch X register to 16 bit

        lda.# 00                ; load bank byte in A
        ldx.# teststr           ; load 16-bit address in X
        jsr.l 00e04e            ; JSL to PUT_STR

        brk 00                  ; break to monitor; 00 is signature byte

  teststr
        .string0 "Mensch"       ; assembler adds final zero byte
        .end
```
Hexdump with 00:2000 as start location for testing with Mensch Monitor:
```
002000: 18 fb e2 20 c2 10 a9 00 a2 11 20 22 4e e0 00 00 
002010: 00 4d 65 6e 73 63 68 00 
```

The manual states the maximum string size is limited to 640 characters. No
arguments are returned, no errors are reported, all registers are preserved.
Though the Manual states that the character is printed to port 3, it will
work fine with the basic USB port.
