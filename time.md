# Time, Date and Timers on the W65C265SXB

The 265SXB comes with eight programmable timers, numbered TO to T7. Of these,
only T7 is only really freely available to the user.

## Timers

| Timer | Name | Function | At boot | Clock |
| --- | :--- | :--- | --- | --- |
| T0 | Watchdog | Breaks infinite loops | off | (TBA) |
| T1 | General purpose | Interrupts for time-of-day clock | on | CLK |
| T2 | (TBA) | Can provide regular interrupts | (TBA) | FCLK/16 |
| T3 | Baud rate generator | Services UARTs | on | (TBA) |
| T4 | Baud rate generator | Serivces UARTs | on | FCLK or P60 |
| T5 | Tone generator | Telephone applications | (TBA) | FCLK |
| T6 | Tone generator | Telephone applications | (TBA) | FCLK |
| T7 | (TBA) | Can provide interrupts  | off | FCLK |

### Timer Usage

All seven timers behave much like their [VIA](https://www.westerndesigncenter.com/wdc/w65c22-chip.php)
or [CIA](https://web.archive.org/web/20181126000922if_/http://archive.6502.org/datasheets/mos_6526_cia_recreated.pdf)
timer cousins. They count downwards from a pre-set value to zero on the falling edge
of the clock, optionally reloading the timer to start the count again. A timer can
optionally generate an interrupt each time it reaches zero. Some of these timers
are capable of sending an interrupt as an output to a physical pin.

Each timer consists of:

- A high and low latch byte, which stores the 16-bit value to count downwards from
- A high and low counter byte, which stores the current value as it moves
towards zero
- A single bit in the Timer Enable Register (TER) to control the timer function
(1 = enabled, 0 = disabled)
- A single bit in the Timer Interrupt Enable Register (TIER) to determine if this
timer should generate an interrupt at zero (1 = enabled, 0 = disabled)
- A single bit in the Timer Interrupt Flag Register (TIFR) to mark which timer
has generated the interrupt. This bit must be cleared to stop the interrupt from
happening again immediately.

#### Timers as implemented on the W65C265SXB

Each timer takes its clock from either the FCLK (3.6868 MHz) or the CLK (32.768 kHz)
oscillators. T4 can count pulses on P60/TIN if not used as a baud rate generator, and
provide square wave reference on P61/TOUT. T7 can generate timer interrupts via TIFR
or used for pulse width measurement (PWM).

While the Mensch Monitor is active in the memory map, the interrupt vector
locations are fixed in memory. Since these are immutable, the code _must_
set up a JMP to a valid interrupt routine, or the entire interrupt
code for that timer should live there. These locations are:

#### Native Mode

| Timer | Vector | Mensch ROM Indirect JMP | Function at Reset |  
| --- | :--- | :--- | :--- |
| T0 | $00:FF80 | $00:0128 | JMP $00:E0B0 (reset routine) |
| T1 | $00:FF82 | $00:0124 | JMP $00:F69F (TOD Clock IRQ) |
| T2 | $00:FF84 | $00:0120 | JMP $00:E0B0 (reset routine) |
| T3 | $00:FF86 | $00:E0B0 | reset routine |
| T4 | $00:FF88 | $00:E0B0 | reset routine |
| T5 | $00:FF8A | $00:E0B0 | reset routine |
| T6 | $00:FF8C | $00:E0B0 | reset routine |
| T7 | $00:FF8E | $00:1C01 | JMP $00:E0B0 (reset routine) |

Note that this has the (possibly) unintended consequence of only allowing
the use of T0, T1, T2, and T7 while the monitor is in the memory map.

## Time-of-Day Clock

The time-of-day clock (TOD) is updated in one-second intervals by software.
There is no RTC chip on the 265SXB. In low-power mode (invoked, for example,
by pressing "x" in the Mensch Monitor), the date and time are not lost, but
they are also _not_ updated.

### Mensch Monitor use

The clock starts up immediately on boot with "12:00:00" as the "current" time.
To see the time and change it, type "t". The Mensch Monitor will prompt you to
type in a new time. Hitting ENTER will keep the old time.

The "current" date is "07-01-93" (note to non-American users: This format is
"Month-Day-Year"). To see the date and change it, type "n". As with the time,
the Monitor will prompt you to change it by typing a similiar string.
