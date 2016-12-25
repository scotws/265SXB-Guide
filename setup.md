# Setting up the 265SXB

Getting the 265SXB up and running requires the board itself, a USB cable, and a
host computer. The board draws power via the USB connection, which is also used
for the terminal. This gives you access to the built-in monitor program.

> Note: Various documents online suggest you need a second, USB-to-TTL cable
> attached to the UART port J5 at the bottom right to access the terminal. This
> is incorrect.

## Connecting a Linux machine

As an example, we connect the 265SXB to a Ubuntu Linux machine via a USB port.

1. Attach the USB cable to the board at J6, the power jack in the bottom middle.
Attach the other end to your computer. This should make the power LED light up.
Feel free to cackle and whisper "It's alive!"

2. On the Ubuntu machine, we will use `putty` as a terminal program. If not
already present, install it with `sudo apt-get install putty` from a shell. To
start the terminal program, you might need to type `sudo putty`.

3. To find out which USB port on the host computer we are using, run `dmesg |
grep tty` from the shell. In our case, the port is `/dev/ttyUSB0`:
```
scot@core:~ dmesg | grep tty
[    0.000000] console [tty0] enabled
[    0.655088] 00:06: ttyS0 at I/O 0x3f8 (irq = 4, base_baud = 115200) is a
16550A
[ 8791.523313] usb 3-6: FTDI USB Serial Device converter now attached to ttyUSB0
```

4. Configure putty: Under the Terminal setting, enable "implicit LF in every
CR". Under the Session setting, select "Serial" and use the port address found
in the previous set as the Serial Line. Click "Open" at the bottom of the window
to create the connection.

![Putty Setup]
(https://github.com/scotws/265SXB-Guide/tree/master/Images/putty_setup_20161225.png
"Putty Setup")

5. On the 265SXB, push the Reset button to the right of the power jack J6. 

This should bring up the Mensch Monitor, a very basic form of an operating
system. See the chapter on [The
Monitor](https://github.com/scotws/265SXB-Guide/blob/master/monitor.md) for more
detail.

Instead of putty, the command-line program `minicom` can be used. See
https://help.ubuntu.com/community/Minicom for details.

