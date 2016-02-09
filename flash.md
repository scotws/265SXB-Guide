# Flash Memory

If you install a SST39SF010A flash memory chip into your 265SXB then you can
develop alternative ROM images that can execute automatically when the board
is powered on.

The SST39SF010A memory chip is writeable under software control. Under normal
operation the chip acts as a standard ROM but by writing an unlock sequence
to specific offsets within the ROM the chip can be made to erase itself
(entirely or in 4K chunks) and store new data.

## Selecting Banks

## Erasing Pages


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


## Writing Bytes


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




# Autobooting ROM Images

During the power on reset processing the Mensch Monitor checks the memory at
00:8000 for the string 'WDC'. If found the monitor jumps to address 00:8004
handing control over to new firmware.

When you are writing a ROM you should either use banks 0-2, which will be
deselected when the reset button is pressed, or only change detection string
to 'WDC' when the code is thoroughly tested.

If you are writing your own ROM with the WDC assembler then the ROM header
is defined as follows:
```
                section offset $8000
                
                db      'WDC',0         ; Only put WDC here when the ROM works
reset:
                nop                     ; Execution starts here
```
In your ROM needs to use hardware interrupts then it must disable the Mensch
ROM to reveal its own interrupt vector table at 00:FFE0.
