# Erratum for the W65C265SXB Manual and Mensch Monitor ROM listing

This text includes known errors, typos, and bugs in the
[Manual](http://www.westerndesigncenter.com/Wdc/documentation/265monrom.pdf) for
the 265SXB and the [code
listing](http://www.westerndesigncenter.com/wdc/documentation/265iromlist.pdf)
of the Mensch Monitor ROM. 

## The Manual

(None registered so far)

## The Mensch Monitor ROM assembler code listing

- Page 4, line 179, in comment: bit 7 selects for banks $C0 to $FF, not "to CF"
- Page 109, lines 4970-4972, in code: CONTROL_TONES pushes Direct Page and Bank
  registers to the stack at the beginning of the routine, but does not pull them
  before exit (missing ```PLB``` and ```PLD``` instructions).


