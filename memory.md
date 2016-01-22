# The 265SXB Memory System

The CPU at the heart of the 265SXB, the 65816, provides 24 address lines and can
address up to 16 MB of memory. Internally, this space is segemented into 256
"banks" of 64k each. When the CPU is emulating the 6502, only the first bank
(first two hex digits are "00", the _zero bank),_ is accessed. See the 65816
Programming Manual for more details.

If this sounds confusing, the memory settings of the 265SXB are worse. The
easist way to understand them is to start with the "out of the box"
configuration and then to work our way through the various ways more addressing
range can be added.

## Out of the box



### The basic memory map


## The on-board 32k RAM

## Adding Flash ROM

## Switching off the built in 8k ROM 
