# Flash Memory

If you install a SST39SF010A flash memory chip into your 265SXB then you can
develop alternative ROM images that can execute automatically when the board
is powered on.

The SST39SF010A memory chip is writeable under software control. Under normal
operation the chip acts as a standard ROM but by writing an unlock sequence
to specific offsets within the ROM the chip can be made to erase itself
(entirely or in 4K sectors) and store new data.

## Selecting Banks

The active 32K region of the 128K flash memory chip can be controlled by pins
connected to port 4. On reset the PD4<4> and PD4<3> are configured as inputs
and external resistors pull the FAMS and FA15 signals high. To access other
banks one or both of the pins should be reconfigured as a low output.

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

## Chip Erase

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

Whenever the flash chip is performing an internal operation a read will
return a value in which bit 7 is the inverse of the real value. During an
erase this means a read will return $7F until the erase is finished.

## Sector Erase

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

## Writing Bytes

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

# Autobooting ROM Images

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
