# A Most Very Unofficial Guide to the W65C265SXB

The W65C265SXB ("265SXB" for short) is an engineering development board
-- basically a single-board computer in the same sense as the Raspberry Pi --
produced the Western Design Center (WDC). It is based on the W65C265S microcontroller,
which in turn has a 65816 MPU at its core -- the 8/16-bit hybrid "big sibling"
of the famous 6502 MPU that powered classic computers such as the VIC-20 and
Apple II. Because of the hybrid nature of the 65816, the 265SXB is pretty much
the simplest way to get started with a 6502 system.

This document is an unofficial guide to setting up, programming, expanding, and
using the 265SXB. It supplements the official documentation provided by WDC.

## Index

### First Steps

[Overview:](https://github.com/scotws/265SXB-Guide/blob/master/overview.md)
What's in the box.  
[Setup:](https://github.com/scotws/265SXB-Guide/blob/master/setup.md) Getting
the 265SXB up and running.  
[First
Programs:](https://github.com/scotws/265SXB-Guide/blob/master/simple_programs.md)
Simple things to do with the 265SXB right out of the box.  
[Uploading
Programs:](https://github.com/scotws/265SXB-Guide/blob/master/entering_code.md)
How to enter and/or upload programs to the 265SXB.  

### Hardware

[Memory:](https://github.com/scotws/265SXB-Guide/blob/master/memory.md) Out of
the box configuration, how chip select signals are generated.   
[Flash:](https://github.com/scotws/265SXB-Guide/blob/master/flash.md) How to
install, select a bank, and erase and program the flash memory chip under
software control.  
[Serial
Lines:](https://github.com/scotws/265SXB-Guide/blob/master/serial_lines.md)
Connecting USB serial adapters to the serial lines of the 265SXB.   
[Clock and Timers:](https://github.com/scotws/265SXB-Guide/blob/master/time.md)
Using the built-in clock functions.

### Software

[WDC Tools:](https://github.com/scotws/265SXB-Guide/blob/master/wdc_tools.md)
Installing and working with the WDC development packages.  
[The Monitor:](https://github.com/scotws/265SXB-Guide/blob/master/monitor.md)
Using the built-in Mensch Monitor program, a miniature operating system.  


### Projects and Expansions

[Expansions:](https://github.com/scotws/265SXB-Guide/blob/master/expansions.md)
Expansion boards, also known as "daughter boards" or shields.

### Appendix

[Link list:](https://github.com/scotws/265SXB-Guide/blob/master/links.md) Useful
links.  
[Erratum:](https://github.com/scotws/265SXB-Guide/blob/master/erratum.md) List
of known errors in the Manual and bugs in the Mensch Monitor ROM listing.

## Thanks

Various members of the [6502.org Forum](http://forum.6502.org/index.php)
contributed to this document and gave invaluable advice.

## Disclaimer 

No liability for the contents of these documents can be accepted. Use the
concepts, examples and information at your own risk. There may be errors and
inaccuracies that could be damaging to your system.  The author(s) take no
responsibility for damage resulting. 

All copyrights are held by their by their respective owners, unless specifically
noted otherwise. Use of a term in this document should not be regarded as
affecting the validity of any trademark or service mark. Naming of particular
products or brands should not be seen as endorsements.

