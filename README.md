pyOCD
=====
pyOCD is an Open Source python 2.7 based library for programming and debugging 
ARM Cortex-M microcontrollers using CMSIS-DAP. Linux, OSX and Windows are supported.

You can use the following interfaces:

1. From a python interpretor:
  * halt, step, resume execution
  * read/write memory
  * read/write block memory
  * read-write core register
  * set/remove hardware breakpoints
  * flash new binary
  * reset

2. From a GDB client, you have all the features provided by gdb:
  * load a .elf file
  * read/write memory
  * read/write core register
  * set/remove hardware breakpoints
  * high level stepping
  * ...

Installation
------------

The latest stable version of pyOCD may be done via
[pip](https://pip.pypa.io/en/stable/index.html) as follows:

```Shell
$ pip install --pre -U pyocd
```

To install the latest development version (master branch), you can do
the following:

```Shell
$ pip install --pre -U https://github.com/mbedmicro/pyOCD/archive/master.zip
```

Note that you may run into permissions issues running these commands.
You have a few options here:

1. Run with `sudo` to install pyOCD and dependencies globally
2. Specify the `--user` option to install local to your user
3. Run the command in a
   [virtualenv](https://virtualenv.pypa.io/en/latest/) local to a
   specific project working set.

Standalone GDB Server
---------------------

When you install pyOCD via pip, you should be able to execute the
following in order to start a GDB server powered by pyOCD:

```Shell
pyocd
```

You can get additional help by running `pyocd --help`.

### Recommended GDB and IDE setup
The GDB server works well with Eclipse and the GNU ARM Eclipse OpenOCD plug-in.
To view register the Embedded System Register Viewer plugin can be used.
These can be installed from inside eclipse using the following links:
GNU ARM Eclipse:  http://gnuarmeclipse.sourceforge.net/updates
Embedded System Register Viewer: http://embsysregview.sourceforge.net/update

The pyOCD gdb server executable will run as a drop in place replacement for 
OpenOCD.  If a supported mbed development board is being debugged the target
does not need to be specified, as pyOCD will automatically determine this.  
If an external processor is being debugged then "-t [processor]" must
be added to the command line.  For more information on setup see [this post for OpenOCD](http://gnuarmeclipse.livius.net/blog/openocd-debugging/)


Examples
--------
### Tests
A series of tests are provided in the test directory:
* basic_test.py: a simple test that checks:
  * read/write core registers
  * read/write memory
  * stop/resume/step the execution
  * reset the target
  * erase pages
  * flash a binary
* gdb_test.py: launch a gdbserver
* gdb_server.py: an enhanced version of gdbserver which provides the following options:
  * "-p", "--port", help = "Write the port number that GDB server will open."
  * "-b", "--board", help="Connect to board by board id."
  * "-l", "--list", help = "List all connected boards."
  * "-d", "--debug", help = "Set the level of system logging output."
  * "-t", "--target", help = "Override target to debug."
  * "-n", "--nobreak", help = "Disable halt at hardfault handler."
  * "-r", "--reset-break", help = "Halt the target when reset."
  * "-s", "--step-int", help = "Allow single stepping to step into interrupts."
  * "-f", "--frequency", help = "Set the SWD clock frequency in Hz."
  * "-o", "--persist", help = "Keep GDB server running even after remote has detached."
  * "-bh", "--soft-bkpt-as-hard", help = "Replace software breakpoints with hardware breakpoints."
  * "-ce", "--chip_erase", help="Use chip erase when programming."
  * "-se", "--sector_erase", help="Use sector erase when programming."
  * "-hp", "--hide_progress", help = "Don't display programming progress."
  * "-fp", "--fast_program", help = "Use only the CRC of each page to determine if it already has the same data."


### Hello World example code
```python
from pyOCD.board import MbedBoard

import logging
logging.basicConfig(level=logging.INFO)

board = MbedBoard.chooseBoard()

target = board.target
flash = board.flash
target.resume()
target.halt()

print "pc: 0x%X" % target.readCoreRegister("pc")
    pc: 0xA64

target.step()
print "pc: 0x%X" % target.readCoreRegister("pc")
    pc: 0xA30

target.step()
print "pc: 0x%X" % target.readCoreRegister("pc")
   pc: 0xA32

flash.flashBinary("binaries/l1_lpc1768.bin")
print "pc: 0x%X" % target.readCoreRegister("pc")
   pc: 0x10000000

target.reset()
target.halt()
print "pc: 0x%X" % target.readCoreRegister("pc")
   pc: 0xAAC

board.uninit()
```

### GDB server example
Python:
```python
from pyOCD.gdbserver import GDBServer
from pyOCD.board import MbedBoard

import logging
logging.basicConfig(level=logging.INFO)

board = MbedBoard.chooseBoard()

# start gdbserver
gdb = GDBServer(board, 3333)
```
gdb server:
```
arm-none-eabi-gdb basic.elf

<gdb> target remote localhost:3333
<gdb> load
<gdb> continue

```

Architecture
------------

### Interface
An interface does the link between the target and the computer.
This module contains basic functionalities to write and read data to and from
an interface. You can inherit from ```Interface``` and overwrite ```read()```, ```write()```,...

Then declare your interface in ```INTERFACE``` (in ```pyOCD.interface.__init__.py```)

### Target
A target defines basic functionalities such as ```step```, ```resume```, ```halt```, ```readMemory```,...
You can inherit from Target to implement your own methods.

Then declare your target in TARGET (in ```pyOCD.target.__init__.py```)

### Transport
Defines the transport used to communicate. In particular, you can find CMSIS-DAP. 
Implements methods such as ```memWriteAP```, ```memReadAP```, ```writeDP```, ```readDP```, ...

You can inherit from ```Transport``` and implement your own methods.
Then declare your transport in ```TRANSPORT``` (in ```pyOCD.transport.__init__.py```)

### Flash
Contains flash algorithm in order to flash a new binary into the target.

### gdbserver
Start a GDB server. The server listens on a specific port. You can then
connect a GDB client to it and debug/program the target

Then you can debug a board which is composed by an interface, a target, a transport and a flash
