# Custom Firmware
Running Custom Firmware on an H1 GSC is not as simple as it might seem at first glance. Assuming you have a way to solder and get it on there, you also have to trick the chip into actually running it. This guide will cover how to do that.

### Be warned!!!!
All existing methods of Custom Firmawre are _tethered_! This means you'll have to redo whatever method you choose each time the GSC resets (for BootCon, you only need to install the firmware once, but the console part you must redo each reset)!


## BootCon Method
First thing's first, you'll want to build the firmware and injectors. You can do this and get all the utilities you need by running `make shaft smiko-cfw` in the root of this repository (make sure you clone with submodules ON). Make sure you install these utilities to your Linux system with `sudo make install`, if you ever wish to remove them, binaries are placed in /usr/local/bin, and firmware in /usr/share/smiko/firmware.

To continue, make sure you're holding H1_BOOT_CONFIG (also known as the dev_mode line on servod) high, and run the following: `sudo shaft -f build/firmware/smiko-bootcon-cfw.bin --bootstrap`. If you're flashing/building on a Rasberry Pi, make sure to pass `-s /dev/spidev#.#` to the correct SPI device.

Once the firmware is installed, now open a CCD console on the target. If you soldered to get UART, you'll want to screen that console, but any old CCD will do. On the target, we first have to prepare some registers first. Cr50 0.0.6 has a much looser console we can use, so we'll use it to set up the chip. You'll likely want to copy and paste commands from this document. Run each of the following commands, in order, on the console:

```
rw 0x40090140 0x10000
rw 0x40090144 0x20
rw 0x40090008 0x7
rw 0x4009028c 0x40000
rw 0x40090290 0x80000
rw 0x40090288 0x3
rw 0x40091004 0xe303ec7a
rw 0x40091008 0x68a03a27
rw 0x4009100c 0xdd18053e
rw 0x40091010 0x39f8dbbd
rw 0x40091014 0x9b553578
rw 0x40091018 0xb4598244
rw 0x4009101c 0xc59f62d1
rw 0x40091020 0x61b8509e
rw 0x40091024 0x0

rw 0x44404
```

This will unlock the GLOBALSEC execution on REGION2, and the last `rw` command will output an address. Copy it, and finally, run the following, with the address in place of "[addr]":
```
sysjump [addr]
```
This will complete the jump to Smiko CFW, which is all the code in HavenOverflow/Cr50 in chip/smiko. You can modify this code however you'd like, and the smiko builder will pack it all together for you. (tip: You can also test all of this in gscemulator without running the initial `rw` commands)



## Extortion Method
This method is a bit more complicated, and a bit more destructive. I wouldn't recommend this method unless you for whatever reason can only locate the UART Tx lines on the board. If you're not running Cr50 RO 0.0.10 or lower, you'll need to use Boostrap to downgrade it, in which case I recommend just using the Bootstrap Method at that point. First, build the required utilities and firmware with `make shaft smiko-cfw` and install them with `sudo make install`.

To get started, begin by putting the H1 in rescue mode. Make sure you have a UART TTY mapped somewhere on /dev/ttyUSB# (this is NOT the same as using a SuzyQ, you _MUST_ solder to UART or the RO channel will not open). Run `sudo shaft -f build/firmware/smiko-bootcon-cfw.bin --rescue --extortion`, and it will automatically flash and run the CFW image for you.