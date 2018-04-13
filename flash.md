# W65C265SXB Flash Memory 

The 265SXB comes with an unpopulated PLCC socket designed to hold a 128K flash
memory chip (70ns or faster; U3 in upper right corner of the board). When
installed, 32K of the flash memory is mapped into the address space between
00:8000 and 00:FFFF. How much is visible depends on whether the mask ROM
(00:E000 - 00:FFFF) and peripheral registers/on-chip RAM (00:DF00 - 00:DFFF) are
enabled. You can switch between the four 32K banks. Also, installing flash
memory allows you to develop alternative ROM images that can execute
automatically when the board is powered on.

For these examples we will use the
[SST39SF010A](http://ww1.microchip.com/downloads/en/DeviceDoc/25022A.pdf) flash
memory chip. In the 265SXB, it is writeable under software control. Under normal
operation the chip acts as a standard ROM but by writing an unlock sequence to
specific offsets within the ROM the chip can be made to erase itself (entirely
or in 4K sectors) and store new data.

## Selecting Banks

The SST39SF010A provides 128K of memory, and comes with 17 address pins, A0 to
A16. Since we only need 32K at any given time for the 265SXB, two of those pins
are used to select which one of the four banks is active. The pins are
controlled by the lines FA15 and FAMS on the 265SXB. FA15 is connected to
address line 15 of the Flash memory, and FAMS to address line 16, which in flash
memory chip documentation is called the "most significant address" (MS) of the
memory chip - hence the names. 

Both pins are connected to port 4, which is controlled by a Data Direction
Register (PDD4, 00:DF24) and the actual Data Register (PD4, 00:DF20). On reset,
the PD4<4> and PD4<3> are configured as inputs and external resistors pull the
FAMS and FA15 signals high. We can can take this to mean that we are accessing
the fourth bank (binary "11", or bank 3) of the flash memory be default. To
access the other banks (0 to 2), one or both of the pins should be reconfigured
as a low output.

The following routine is used in the W65C265 version of the SXB-Hacker to set
the pins for a specific bank.

```
; Select the flash ROM bank indicated by the two low order bits of A. The pins
; should be set to inputs when a hi bit is needed and a low output for a lo bit.

                public RomSelect
RomSelect:
                php                             ; Save register sizes
                sep     #1<<5                   ; Then make A 8-bits
                longa   off
                and     #$03                    ; Strip out bank number
                asl     a                       ; And rotate into bits
                asl     a
                asl     a
                pha                             ; Save bit pattern
                eor     #$18                    ; Invert to get directions
                eor     PDD4                    ; Work out change
                and     #$18
                eor     PDD4                    ; And apply to direction reg
                sta     PDD4
                pla
                eor     PD4                     ; Then adjust data register
                and     #$18
                eor     PD4
                sta     PD4
                plp                             ; Restore register sizes
                rts                             ; Done
```

## Writing and Erasing Flash Memory

The in-place write and erase operations of the SST39SF010A are controlled by
command sequences that are sent to specific addresses with specific data. These
sequences only use the 15 address lines A0 to A14, which allows us to specifiy
that we want the high half of the 265SXB Bank 0 address through A15.

> In the following code, we make this clear using the notation of
> ```$8000+$5555``` and ```$8000+2aaa``` for the addresses $5555 and $2AAA
> required.

For the complete list of command sequences, see the [SST39SF010A
documentation.](http://ww1.microchip.com/downloads/en/DeviceDoc/25022A.pdf)

Whenever the flash chip is performing an internal operation, a read will
return a value in which bit 7 is the inverse of the real value. During an erase,
for instance, this means a read will return $7F until the erase is finished.

### Chip Erase

The SST39SF010A provides a Chip-Erase operation, which allows the user
to erase the entire memory array to the '1s' state. This is useful when the
entire device must be quickly erased.

The Chip-Erase operation is initiated by executing a six- byte Software Data
Protection command sequence with Chip-Erase command ($10) with address $5555
in the last byte sequence.

```
                lda     #$aa                    ; Unlock flash
                sta     $8000+$5555
                lda     #$55
                sta     $8000+$2aaa
                
                lda     #$80                    ; Signal erase
                sta     $8000+$5555
                lda     #$aa
                sta     $8000+$5555
                lda     #$55
                sta     $8000+$2aaa
                
                lda     #$10                    ; Chip erase
                sta     $8000+$5555
EraseWait:
                lda     $8000                   ; Wait for erase to finish
                cmp     #$FF
                bne     EraseWait
```

The chip erase operation takes around 70 mSec to complete. The controlling
application should either perform a timed delay or poll the chip.

### Sector Erase

The Sector-Erase operation allows the system to erase the device on a
sector-by-sector basis. The sector architecture is based on uniform sector size
of 4K bytes.

The Sector-Erase operation is initiated by executing a six-byte command load
sequence for Software Data Protection with then Sector-Erase command ($30) as
the last byte.

The flash chip uses the address that the command byte is written to to determine
which 4K region of the chip is erased.

The following code shows how to erase a block determined by an address between
00:8000 and 00:FFFF held in a zero page based pointer. The starting address of the
4K sector will be derived from the high order bits of the address (e.g. ADDR &
$F000) plus the FAMS and FA15 signals.

```
                lda     #$aa                    ; Unlock flash
                sta     $8000+$5555
                lda     #$55
                sta     $8000+$2aaa
                
                lda     #$80                    ; Signal erase
                sta     $8000+$5555
                lda     #$aa
                sta     $8000+$5555
                lda     #$55
                sta     $8000+$2aaa
                
                lda     #$30                    ; Sector erase
                sta     (ADDR)
EraseWait:
                lda     (ADDR)                  ; Wait for erase to finish
                cmp     #$FF
                bne     EraseWait
```

Erasing a sector takes around 18 mSec so further accesses to the flash memory
must wait until the operation completes. 

### Writing Bytes

The flash chip can be programmed on a byte-by-byte basis. The write operation
can only store '0' bits into the memory so the region to be programmed must be
erased prior to writing.

The programming application must perform three write operations to unlock the
memory and initiate a byte write operation followed by a further write to
store the actual data byte.

The following code shows how to write a value held in a RAM location called
DATA at an address in the 00:8000 to 00:FFFF range held in a zero page based
pointer.

```
                lda     #$aa                    ; Yes, unlock flash
                sta     $8000+$5555
                lda     #$55
                sta     $8000+$2aaa
                
                lda     #$a0                    ; Start byte write
                sta     $8000+$5555
                
                lda     DATA                    ; Write the value
                sta     (ADDR)
WriteWait:
                cmp     (ADDR)                  ; Wait for write
                bne     WriteWait
```

A write byte operation takes around 14 uSec. As with the erase operations
you can perform read operation on the memory to find out when the write has
been completed.


## Switching off the built-in 8K ROM 

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
(Images/W65C256SXB_Memory_2.png)

_Image: Bank 00 showing the three simplest memory configurations. Light: RAM,
dark: ROM, red: I/O registers and 64 byte on-chip RAM, grey: empty._

## Autobooting ROM Images

During the power on reset processing the Mensch Monitor checks the memory at
00:8000 for the string 'WDC'. If found the monitor jumps to address 00:8004
handing control over to new firmware.

When you are writing a ROM you should either use banks 0-2, which will be
deselected when the reset button is pressed, or only change the detection string
to 'WDC' when the code is thoroughly tested.

If you are writing your own ROM with the WDC assembler then the ROM header
is defined as follows:
```
                section offset $8000
                
                db      'WDC',0         ; Only put WDC here when the ROM works
reset:
                nop                     ; Execution starts here
```
In your ROM needs to use hardware interrupts or the memory area between 00:C000
and 00:FFFF then it must disable the Mensch Monitor to reveal its own interrupt
vector table at 00:FFE0.

If you load a corrupt firmware image into bank 3 of the ROM then you may need to
remove the flash chip and erase it in an external programmer.
