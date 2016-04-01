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

T4 can count pulses on P60/TIN if not used as a baud rate generator, and
provide square wave reference on P61/TOUT.

T7 can generate timer interrupts via TIFR or used for pulse width measurement
(PWM). 


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

