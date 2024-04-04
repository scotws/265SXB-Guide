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
Manual](http://www.westerndesigncenter.com/wdc/documentation/265monrom.pdf),
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

**GET_CHR** ("get character", 00:E036, main code at 00:FBF9) - Get a single
character from the console, blocking until the character is received. Note this
routine must be called with a long subroutine jump as ```JSL 00E036```. Returns
with the carry bit clear and the character received in 8-bit A register. The
character is not echoed, use GET_PUT_CHR for that. 
```
   ; GET_CHR example program. All numbers are hex. Assumes native mode

        .origin 2000            ; start at 00:2000
        .mpu 65816              ; tells assembler this is a 65816 MPU

        sep 20                  ; switch A register to 8 bit
        rep 10                  ; switch X register to 16 bit

        jsr.l 00:e036           ; JSL to GET_CHR

        brk 00                  ; break to monitor, 00 is signature byte
```
Hexdump with 00:2000 as start location for testing with Mensch Monitor:
```
002000: e2 20 c2 10 22 36 e0 00 00 00 
```
> When you test this program with ```j 002000``` from the Mensch Monitor, it 
> will start waiting for the character immediately after the final 0 of the
> address, because the Monitor command does not require a final ENTER. If you
> do hit ENTER (common enough when first experimenting with the Mensch 
> Monitor), this will put $0D in the A register, the ASCII code for Carriage 
> Return (CR).

If you are using GET_CHR to grab entries, note that things are not as simple as
just getting one byte. ```Arrow Up```, ```Arrow Down```, ```ESC```, ```Page
Up```, ```DEL```, and ```INS``` all return $1B with the carry bit clear. This is
the [ASCII code for Escape](http://www.ascii-code.com/) that is followed by
other ASCII characters for VT100 terminals for certain special characters. If
you are going to use GET_CHR as a base for a command line input loop, you'll
either have to include a timed loop to catch the other characters - see [this
discussion](http://forum.6502.org/viewtopic.php?f=1&t=3552&p=49473#p49473) - or
make due with the CONTROL combinations known from the [Bash shell keyboard
shortcuts](http://www.howtogeek.com/howto/ubuntu/keyboard-shortcuts-for-bash-command-shell-for-ubuntu-debian-suse-redhat-linux-etc/):

Input  | GET_CHR code | ASCII of code | Bash function
------ | ------------ | ------------- | --------------
CTRL a | $01 | Start of Heading (SOH) |  move cursor to start of line ("home")
CTRL b | $02 | Start of Text (STX) |  move cursor left
CTRL c | $03 | End of Text (ETX)   |  abort current process
CTRL d | $04 | End of Transmission (EOT) |  exit current shell
CTRL e | $05 | Enquiry (ENQ) |  move cursor to end of line ("end")
CTRL f | $06 | Acknowledgement (ACK) |  move cursor right
CTRL h | $08 | Back Space (BS) |  rubout character ("backspace")
CTRL l | $0c | Form Feed (FF) |  clear screen
CTRL n | $0e | Shift Out (SO) |  next command ("down")
CTRL p | $10 | Data Line Escape (DLE) |  previous command ("up")
CTRL z | $1a | Substitute (SUB) | suspend process

These all return the carry bit clear. 

**GET_STR** ("get string", 00:E03F, main code at 00:F3D7) - Receives a string
from the console until either ENTER or ESC is received and stores the string
with a terminating 00 byte to a specified buffer. This routine must be called
with a long subroutine jump as ```JSL 00E03F```. The address of the buffer must
be provided by the bank byte in the 8-bit A register and the rest of the address
in 16-bit X register. No arguments are returned. If the string entry was ended
by an ENTER, the carry bit will be clear. If it was ended by an ESC, the carry
bit will be set. Calls GET_PUT_CHR to echo the string.
```
  ; GET_STR example program. All numbers are hex. Assumes native mode

        .origin 2000        ; starts at 00:2000
        .mpu 65816          ; tells assembler this is a 65816 MPU

        clc                 ; make sure we're in native mode
        xce                      
        sep 20              ; switch A register to 8 bit
        rep 10              ; switch X register to 16 bit

        lda.# 00            ; buffer bank is 00
        ldx.# 3000          ; buffer address is 3000
        jsr.l 00:e03f       ; JSL to GET_STR

        brk 00              ; break to monitor, 00 is signature byte
```
Hexdump with 00:2000 as start location for testing with Mensch Monitor:
```
002000: 18 fb e2 20 c2 10 a9 00 a2 00 30 22 3f e0 00 00 
002010: 00 
```
Note that if the string was terminated with ENTER, the last character in the
string before the terminating 00 will be the ASCII character for Carriage Return
(CR, $0D). In the above example, entering "Mensch Monitor" gives us
```
003000: 4D 65 6E 73 63 68 20 4D 6F 6E 69 74 6F 72 0D 00
```
CONTROL-C will not abort the input, but will result in the ASCII character $03
(end of text) being saved; a ENTER is still required to terminate the input. A
single ESC will produce whitespace to the screen while testing in the Mensch
Monitor and requires a second ESC to terminate. Terminating input with ESC will
store everything up to, but not including, the ESC ASCII character in the
buffer, will set the carry flag, and will store 00 in the A register. However,
neither a carriage return (CR) nor the terminating zero are saved.

**GET_BYTE_FROM_PC** ("get byte from input buffer", 00:E033, main code at
00:FBF9) - Returns a character from the input buffer. This routine must be
called with a long subroutine jump as ```JSL 00E033```. If there is data in the
buffer, it is returned with the carry bit clear and the data in the 8-bit
accumulator. Otherwise, carry bit is set, and 8-bit accumulator contains 00
(no data) or 80 (CONTROL-C or ESC received). This routine does not block, so
testing it with the Mensch Monitor runs straight through, returning 00 in the A
register and a set carry bit.

**GET_PUT_CHR** ("get a character and echo to console", 00:E03C, main code at
00:F1C9) Gets a character from the input buffer and echos it back. This routine
must be called with a long subroutine jump as ```JSL 00E03C```. Returns with the
received character in 8-bit A register and the carry bit clear. 
```
   ; GET_PUT_CHR example program. All numbers are hex. Assumes native mode

        .origin 2000            ; start at 00:2000
        .mpu 65816              ; tells assembler this is a 65816 MPU

        clc                     ; make sure we're in native mode
        xce     
        sep 20                  ; switch A register to 8 bit
        rep 10                  ; switch X register to 16 bit

        jsr.l 00:e03c           ; JSL to GET_PUT_CHR

        brk 00                  ; break to monitor, 00 is signature byte
```
Hexdump with 00:2000 as start location for testing with Mensch Monitor:
```
002000: 18 fb e2 20 c2 10 22 3c e0 00 00 00 
```
This routine blocks while waiting. Hitting ESC will return the equivalent ASCII
character $1B with the carry bit clear during testing with Mensch Monitor (where
two Escapes are required). 

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
(00, ASCII "null"). This routine calls PUT_CHR. The routine does not add a
carriage return and/or form feed. 
```
  ; PUT_STR example program. All numbers are hex
        .origin 2000            ; start at 00:2000
        .mpu 65816              ; tells assembler this is a 65816 MPU

        clc                     ; make sure this is native mode
        xce
        sep 20                  ; switch A register to 8 bit
        rep 10                  ; switch X register to 16 bit

        lda.# 00                ; load bank byte in A
        ldx.# teststr           ; load 16-bit address in X
        jsr.l 00e04e            ; JSL to PUT_STR

        brk 00                  ; break to monitor; 00 is signature byte

  teststr
        .byte "Mensch",0 
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

**SEND_CR** ("send carriage return", 00:E066, main code at 00:F1D6) - Prints a
Carriage Return (CR, $0D) to screen. Simply puts $0D in the A register and calls
PUT_CHR.
