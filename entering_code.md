# Entering and Uploading Code to the W65S265SXB

There various ways to get your own programs on the 265SXB. We assume here that
you've written them on one of the various assemblers for the 65816 and have
produced an assembled binary machine code file.  

As an example, we'll use the [PUT_STR
code](https://github.com/scotws/265SXB-Guide/blob/master/monitor.md) from the
chapter on the Mensch Monitor, which calls the Put String subroutine to print
the string "Mensch" on the screen. The resulting code starts at 00:2000 and has
the following hex dump: 
```
002000: 18 fb e2 20 c2 10 a9 00 a2 11 20 22 4e e0 00 00 
002010: 00 4d 65 6e 73 63 68 00 
```
The code size is 24 bytes.


## The hard way: Entering code by hand

We have already seen the simplest, but also hardest way to enter code: Type it in
through the Monitor by hand, as we did with our [first simple
program](https://github.com/scotws/265SXB-Guide/blob/master/simple_programs.md).
Once you have the Monitor started, press the "m" key, enter ```002000`` as the
starting address, and type in the above sequence of bytes. To run the program,
press the "j" key and enter ```002000``` as the start address. 


## The less hard way: Sending S-Records

This is doable for very, very short programs, but anything longer will quickly
become error-prone and most tedious. Fortunately, the Monitor will accept
programs sent to it in a special file format called SREC, the [Motorola
S-record](https://en.wikipedia.org/wiki/SREC_(file_format)) format. Because of
its 65816 MPU core with a 24-bit address space, we need "S28"-style formats.

Converted to S28, our program would be an ASCII file (mensch.s28) with the
following contents:
```
S0220000687474703A2F2F737265636F72642E736F75726365666F7267652E6E65742F1D
S21C00200018FBE220C210A900A21120224EE00000004D656E73636800B2
S5030001FB
S804002000DB
```

We won't go into the format in greater detail, but note that the first to
characters of each line are the "Record Field". We need S2 fields for the data
because of 24 bit addresses, and in the last line S8 for the 24-bit terminating
address. The first S0 line is the "header" and may look differently; in this
case, it is the web page of the tool used to create the S28 file (see below).

There are various ways to create and transmit S28 files. For instance,
Andrew Jacobs' Java-based Assembler at
[http://www.obelisk.me.uk/dev65/](http://www.obelisk.me.uk/dev65/) supports
S-records, see
[http://forum.6502.org/viewtopic.php?f=4&t=3551&p=42505#p42483](http://forum.6502.org/viewtopic.php?f=4&t=3551&p=42505#p42483)
for a discussion on this topic. 


### WDC Tools

The WDC Tool package allows uploading of S28 records through the Debugger. As
described
[elsewhere](https://github.com/scotws/265SXB-Guide/blob/master/wdc_tools.md),
the tools only run on Windows.


### Linux 

Creating and sending S-records with a Linux is a bit more complicated. First, we need the
[srec_cat](http://srecord.sourceforge.net/man/man1/srec_examples.html) program
to create the S28 files. On Ubuntu, run
```
sudo apt-get install srecord
```
To convert a binary file such as our mensch.bin, we need to provide an
**offset** - where the program will be located in memory - and the **execution
start address** for the correct format. In our case, these are the same
addresses. Our complete srec_cat line is:
```
srec_cat mensch.bin -binary -offset 0x2000 -o mensch.s28 -address-length=3 -execution-start-address=0x2000
```

The options "-binary" tell srec_cat that we have a pure binary file,
"-address-length=3" forces the S28 format for 24-bit addresses. To make sure we
have the correct format, use ```srec_info mensch.s28``` to inspect the contents:
```
Format: Motorola S-Record
Header: "http://srecord.sourceforge.net/"
Execution Start Address: 00002000
Data:   2000 - 2017
```
Note the useless Header field.

Next, we can then use a program such as minicom to send the file as "plain
ASCII" to the 265SXB. To do this, press the "s" key in the monitor. Then in
minicom, type ```CONTROL-A Z``` for the main menu, then type "s" to send a file,
using ASCII as a protocol.


## Further ways of entering code

If you are Chuck Norris, the code will enter itself.

