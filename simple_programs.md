# A Simple First Program

Now that the 265SXB is set up and running, we'll prove that it's alive with a
few simple programs.

## Printing a single character

In this first example, we're going to print the letter "a" on the screen. Get
the 265SXB started and the Mensch Monitor running. Now, press the "m" key and
enter the address ```002000```. The Monitor will print a string of hex digits
and then prompt you to add the new values. Type the following sequence (without
spaces, which the Monitor will take care of for you):
```
a9 61 22 4b e0 00 00
```
Hit ENTER when you are done. Now, press the "j" key and type ```002000```, the
address of the little program you just entered. Immediately after typing the
last "0", a small "a" will appear after it and the Monitor will print out the
register contents and other status information. The whole sequence should look
something like this (look for the little "a" right behind the address
```00:2000``` right about in the middle):
```
>m
BB:AAAA 00:2000
Address 0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  

00:2000 F7 FD BD 93 B4 B4 BE 20 DB BE 1A 3D C7 DC B1 A0 
00:2000 a9 61 22 4b e0 00 00 

READY
>j  
Enter Address  BB:AAAA 00:2000a

PCntr     Acc    Xreg   Yreg   Stack
00:2008   00 61  E0 B7  00 FE  01 FC  

  DirRg  F  DBk
  00 00  20 00  


Status Reg
N  V  M  X  D  I  Z  C
0  0  1  0  0  0  0  0  

>
```
The little program we entered by hand in machine language looks like this in 
assembler:
```
Standard Assembler:                  Typist's Assembler:

    LDA #$61                            lda.# 61
    JSL 00E04B                          jsr.l 00:e04b
    BRK                                 brk
```
What we are doing here is using the Mensch Monitor's built-in PUT_CHR routine
to print the character in the Accumulator to the screen. More about using the
built-in routines in the [Monitor chapter]
(https://github.com/scotws/265SXB-Guide/blob/master/monitor.md).

