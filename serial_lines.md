# Serial Lines on the W65C265SXB

The 265SXB has four built-in UARTs for serial connections, numbered 0 to 3. The
Mensch Monitor program uses UART3 with 9600 baud through the power jack in J6 at
the bottom of the board for basic connections out of the box. Access to the
other serial lines is through J5 on the bottom right of the board, where Ground
(GND), 5V Power, and the four transmit (TXD)/receive (RXD) pairs are exposed.

## Hardware 

You will usually want to use a USB serial adapter such as a PL2303
module. You will need to connect GND, TXD, and RXD to the correct pins of J5.

### J5 Header

To find the correct pins, note that the *bottom right* pin of J5 is marked with
a dot as pin 1 (5V). The one to the left of it is pin 2 (GND).

![J5 Header Pins]
(https://github.com/scotws/265SXB-Guide/blob/master/Images/W65C265SXB_J5_Header_Pins.png)

To connect the USB serial adapter, connect ground to ground and the TXDs to
RXDs.

![USB serial adapter]
(https://github.com/scotws/265SXB-Guide/blob/master/Images/265SXB-uart0-usb.jpg)

_Image: USB serial adapter connected to UART0 through pins 3 and 4 with ground
on pin 2. 265SXC with [1 MB
Shield](https://github.com/scotws/265SXB-Guide/blob/master/expansions.md). Note
the dot next to pin 1 on the header._

Put differently, the the TXD pins are on the left, the RXD pins on the right. 

### Registers 

The four UARTs each have a data register (formal name: _asynchronous
receiver/transmitter register,_ ARTDx) and a _status/control_ register
_(asynchronous status/control register,_ ACSRx). They are located at the
following memory addresses:

| UART (x) | Data Reg (ARTDx) | Stat/Cont Reg (ACSRx) | 
| :------: | :--------------: | :-------------------: | 
|  0       | 00:DF71          |  00:DF70              | 
|  1       | 00:DF73          |  00:DF72              |
|  2       | 00:DF75          |  00:DF74              |
|  3       | 00:DF77          |  00:DF76              |

The control/status registers have the following bits: 

| Bit | Function           |   
| :-: | :----------------- |
|  0  | Xmit port enable   |
|  1  | Xmit IRQ source    |
|  2  | 7/8 bit data       |
|  3  | Parity enable      |
|  4  | Odd/even parity    |
|  5  | Receive enable     |
|  6  | Software semiphore |
|  7  | Receive error flag |

In the Mensch Monitor, the serial parts are initialized starting at 00:FE71, and
the Baud Rate is set up starting 00:FE21. 

