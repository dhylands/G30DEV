This is a MicroPython board definition file for the G30DEV from ghielectronics.

The produce page for the G30DEV can be found here: https://www.ghielectronics.com/catalog/product/555

Build the firmware using:
```
cd micropython/stmhal/boards
git clone https://github.com/dhylands/G30DEV.git
cd ..
make BOARD=G30DEV
```

The G30DEV board can be powered via USB, or by providing power through the
provided barrel jack.

To get the device in DFU mode, you need to connect B0, which is the test
point located on the bottom edge of the board, immediatley below the text "G30 SoC"
(between R3 and R7), to 3.3v, while powering up or RESETing the device. I used
the 3.3v available on the black expansion header.

Once in DFU mode, you should be able to use the ```dfu-util --list``` command and see
something like this:
```
Found DFU: [0483:df11] ver=2200, devnum=14, cfg=1, intf=0, alt=3, name="@Device Feature/0xFFFF0000/01*004 e", serial="377A364D3234"
Found DFU: [0483:df11] ver=2200, devnum=14, cfg=1, intf=0, alt=2, name="@OTP Memory /0x1FFF7800/01*512 e,01*016 e", serial="377A364D3234"
Found DFU: [0483:df11] ver=2200, devnum=14, cfg=1, intf=0, alt=1, name="@Option Bytes  /0x1FFFC000/01*016 e", serial="377A364D3234"
Found DFU: [0483:df11] ver=2200, devnum=14, cfg=1, intf=0, alt=0, name="@Internal Flash  /0x08000000/04*016Kg,01*064Kg,03*128Kg", serial="377A364D3234"
```

The G30DEV is read protected. In order to flash MicroPython on it, you need
to unprotect it, which will in turn erase the part. This is a one-time
operation. Once MicroPython is flashed, the device will remain unprotected.

Unprotect/Erase the device using:
```
dfu-util -s :unprotect:force -a 0 -d 0483:df11 -D build-G30DEV/firmware.dfu
```

Now you can flash MicroPython using:
```
make BOARD=G30DEV deploy
```

Reboot with BOOT0 unconnected, and it should boot up MicroPython on the REPL:
```
MicroPython v1.8.2-15-g6381d71 on 2016-07-15; G30DEV with STM32F401CE
Type "help()" for more information.
>>> 
```

Now that MicroPython is installed, as long as you can get to the REPL, then you
can get back into DFU mode by using the command:
```
>>> pyb.bootloader()
```
or by connecting the B0 test point to 3.3v.

LDR0 is connected to the User Switch.

## Peripheral Testing status:

Tested:
- USB Repl/Mass Storage
- LED (including PWM intensity)
- SD Card
- LDR0/User Switch
- LDR1
- JP1
- Piezo Buzzer
- LCD
- LCD Backlight
- RTC Clock (board needs to be initially powered on for a few minutes)
- COM1 (UART1)
- COM2 (UART2)

### Notes on COM2 (DB9)

There is no ground on pin 5. It's connected to the frame ground instead.
So you really need to ensure that you have access to the frame ground. I had
to test with a wire plugged into one of the GND signals on the expansion
connector in order to get a reliable ground.

If you unscrew the mounting bolts on the connector then your frame ground doesn't
go through to the shell, and my DB9 breakouts were also the same way. So I had to
remove one set of mounting connectors in order for the the connectors to plug
it in and then you lose the frame ground. If you're planning on using this
connector, I'd recommend soldering a small wire (on the back of the PCB) from
pin 5 on the DB8 connector to the mounting solder connection.

## Not yet tested:
- WizNet 5500 Ethernet
