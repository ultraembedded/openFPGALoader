# openFPGALoader
Utility for programming Intel/Altera Cyclone Xilinx Serie 7 and Lattice MachXO3

__Current support:__

* Trenz cyc1000 Cyclone 10 LP 10CL025 (memory and spi flash)
* Digilent arty Artix xc7a35ti (memory and spi flash)
* Lattice MachXO3LF Starter Kit LCMX03LF-6900C (flash)
* [Trenz Gowin LittleBee (TEC0117)](https://shop.trenz-electronic.de/en/TEC0117-01-FPGA-Module-with-GOWIN-LittleBee-and-8-MByte-internal-SDRAM)

__Supported FPGA:__

* Gowin [GW1N (GW1NR-9)](https://www.gowinsemi.com/en/product/detail/2/) (SRAM
  only)

__Supported cables:__

* JTAG-HS3: jtag programmer cable from digilent
* FT2232: generic programmer cable based on Ftdi FT2232

## compile and install

This application uses **libftdi1**, so this library must be installed (and,
depending of the distribution, headers too)
```bash
apt-get install libftdi1-2 libftdi1-dev libftdipp1-3 libftdipp1-dev libudev-dev
```
and if not already done, install **pkg-config**, **make** and **g++**.

To build the app:
```bash
$ make
```
To install
```bash
$ sudo make install
```
Currently, the install path is hardcoded to /usr/local

## Usage

```bash
openFPGALoader --help
Usage: openFPGALoader [OPTION...] BIT_FILE
openFPGALoader -- a program to flash cyclone10 LP FPGA

  -b, --board=BOARD          board name, may be used instead of cable
  -c, --cable=CABLE          jtag interface
  -d, --device=DEVICE        device to use (/dev/ttyUSBx)
  -o, --offset=OFFSET        start offset in EEPROM
  -r, --reset                reset FPGA after operations
  -v, --verbose              Produce verbose output
  -?, --help                 Give this help list
      --usage                Give a short usage message
  -V, --version              Print program version

```
To have complete help

### Generic usage

#### display FPGA

With board name:
```bash
openFPGALoader -b theBoard
```

With cable:
```bash
openFPGALoader -c theCable
```

With device node:
```bash
openFPGALoader -d /dev/ttyUSBX
```

**Note:** for some cable (like *digilent* adapters) signals from the converter
are not just directly to the FPGA. For this case, the *-c* must be added.

**Note:** when -d is not provided, *openFPGALoader* will opens the first *ftdi*
found, if more than one converter is connected to the computer,
the *-d* option is the better solution

#### Reset device

```bash
openFPGALoader [options] -r
```

#### load bitstream device (memory or flash)
```bash
openFPGALoader [options] /path/to/bitstream.ext
```

### CYC1000

#### loading in memory:

sof to svf generation:
```bash
quartus_cpf -c -q -g 3.3 -n 12.0MHz p project_name.sof project_name.svf
```
file load:
```bash
openFPGALoader -b cyc1000 project_name.svf
```

#### SPI flash:
sof to rpd:
```bash
quartus_cpf -o auto_create_rpd=on -c -d EPCQ16A -s 10CL025YU256C8G project_name.svf project_name.jic
```
file load:
```bash
openFPGALoader -b cyc1000 -r project_name_auto.rpd
```

**Note about SPI flash:
svf file used to write in flash is just a bridge between FT2232 interfaceB
configured in SPI mode and sfl primitive used to access EPCQ SPI flash.**

**Note about FT2232 interfaceB:
This interface is used for SPI communication only when the dedicated svf is
loaded in RAM, rest of the time, user is free to use for what he want.**

### ARTY

To simplify further explanations, we consider the project is generated in the
current directory.

#### loading in memory:

*.bit* file is the default format generated by *vivado*, so nothing special
task must be done to generates this bitstream.

__file load:__
```bash
openFPGALoader -b arty *.runs/impl_1/*.bit
```

#### SPI flash:
.mcs must be generates through vivado with a tcl script like
```tcl
set project [lindex $argv 0]

set bitfile "${project}.runs/impl_1/${project}.bit"
set mcsfile "${project}.runs/impl_1/${project}.mcs"

write_cfgmem -format mcs -interface spix4 -size 16 \
    -loadbit "up 0x0 $bitfile" -loaddata "" \
    -file $mcsfile -force

```
**Note:
*-interface spix4* and *-size 16* depends on SPI flash capability and size.**

The tcl script is used with:
```bash
vivado -nolog -nojournal -mode batch -source script.tcl -tclargs myproject
```

__file load:__
```bash
openFPGALoader -b arty *.runs/impl_1/*.mcs
```
### MachXO3 Starter Kit

#### Flash memory:

*.jed* file is the default format generated by *Lattice Diamond*, so nothing
special must be done to generates this file.

__file load__:
```bash
openFPGALoader -b machXO3SK impl1/*.jed
```

### Trenz GOWIN LittleBee (TEC0117)

#### Flash SRAM:

*.fs* file is the default format generated by *Gowin IDE*, so nothing
special must be done to generates this file.

__file load__:
```bash
openFPGALoader -b machXO3SK impl/pnr/*.fs
```
