# AMICO - Another Micro 8-bit Computer
This is a home project for a simple 8-bit microcomputer, based on a "virtual CPU" implemented by a programmed microcontroller (as a PIC or similar).

![amico-mercury.jpg](amico-mercury.jpg)



## Features
- 8-bit Data BUS
- 16-bit Address BUS
- Memory mapped I/O
- time based or signal based operation


## Architecture
![amico-architecture.jpg](amico-architecture.jpg)

### BUS

* Power lines :
	* +5V
	* Gnd
* Address lines :
	* A0-A15 - 16-bit Address
* Data lines :
	* D0-D7 - 8-bit Data
* Control lines :
    * /READ - read operation. "Lo" value indicates the device is being read by the CPU.
	* /WRITE - write operation. "Lo" value indicates the device is being written by the CPU.
    * /WAIT - wait for slow device operation. "Lo" value puts the CPU into IDLE mode until line come back "Hi".
    * /RESET - system reset. "Lo" value causes a system reset.
    * BCLK - bus clock. Reference clock for the bus operations.
* Auxiliary lines:
    * AUX0 or BCLK/n - auxiliary line 0 or programmable clock (BCLK divided by n). Available and programmable by any device.
    * AUX1 - auxiliary line 1. Available for custom usage by any device.
    * AUX2 - auxiliary line 2. Available for custom usage by any device.


### Address Map
The memory and registers of the I/O devices are mapped on the same address space (memory mapped I/O), so a memory address may refer to either a portion of physical RAM/ROM or to registers of the I/O device. The 64KB of accessible addresses are logically divided by 8 blocks of 8KB each. The address map of the AMICO system looks like this:

| Addresses     | Size | Assignment           
|---------------|------|----------------------
| C000H - FFFFH | 16KB | USER ROM (banks 0,1 for 8KB each) 
| A000H - BFFFH |  8KB | SYSTEM ROM           
| 8000H - 9FFFH |  8KB | I/O (devices 0..7 for 1KB each)   
| 0000H - 7FFFH | 32KB | RAM (banks 0..3 for 8KB each)     


### Address MAP Decoder
#### RAM Address Decoder
The address line A15=LOW selects all the 32KB RAM space, then no gate ports are needed.

#### ROM Address Decoder
The 8KB of system ROM can be accessed with the address lines A15=HIGH and A14=LOW. Then two gates (NAND) are needed to implements the logic, or a single demux (138 or 139).

#### I/O Address Decoder
To access to the 8 x 1KB I/O space, a demux (138) is needed, with address lines A13, A14, A15 selcting I/O devices 1 through 8, and the upper address lines set as: A13=A14=LOW and A15=HIGH.


### Read/Write Operations
Fast enought Read/Write operations (inside a bus clock period) are time based sequences of command, generated from the CPU to the target device. The CPU sets the address and/or data lines on the rising edge of the bus clock signal, and reads/holds the data on/until the next rising edge.

Via the *WAIT/* control line, slow devices can perform operations in signal-based mode manner.


#### Write Operation
1. CPU writes and latches Address on BUS
2. CPU writes Data on BUS
3. CPU sets Control Line */WRITE*
4. DEVICE latches Data from BUS
5. CPU unsets Control Line */WRITE*

```text
                  __     __     __     __     __   
BCLK          ___|  |___|  |___|  |___|  |___|  |__
                        T1    T2
CPU WRITE                 ____________ 
A0-A15        -----------<____________>------------

CPU WRITE                   ____ 
D0-D7         -------------<____>------------------

CPU SET       ____________       __________________
/WRITE                    |_____|

```

#### Read Operation
1. CPU writes and latches Address on BUS
2. CPU sets Control Line */READ*
3. DEVICE writes Data on BUS
4. CPU reads Data from BUS
5. CPU unsets Control Line */READ*

```text
                  __     __     __     __     __   
BCLK          ___|  |___|  |___|  |___|  |___|  |__
                        T1    T2
CPU WRITE                 ____________ 
A0-A15        -----------<____________>------------

CPU SET       ____________        _________________
/READ                     |______|

CPU READ                        _
D0-D7         -----------------<_>-----------------

```

#### Slow Device I/O Write Operation
1. CPU writes and latches Address on BUS
2. CPU writes Data on BUS
3. CPU sets Control Line */WRITE*
4. DEVICE sets Control Line */WAIT*
5. CPU enters into IDLE mode
6. DEVICE latches Data from BUS
7. DEVICE unsets Control Line */WAIT*
8. CPU unsets Control Line */WRITE*

```text
                  __     __     __     __     __   
BCLK          ___|  |___|  |___|  |___|  |___|  |__
                        T1    T2            Tn
CPU WRITE                 ________________________ 
A0-A15        -----------<________________________>

DEVICE SET    ____________                 ________
/WAIT                     |_______________|

CPU WRITE                   __________________ 
D0-D7         -------------<__________________>----

CPU SET       ____________                     ____
/WRITE                    |___________________|

```

#### Slow Device I/O Read Operation
1. CPU writes and latches Address on BUS
2. CPU sets Control Line */READ*
3. DEVICE sets Control Line */WAIT*
4. CPU enters into IDLE mode
5. DEVICE writes Data on BUS
6. DEVICE unsets Control Line */WAIT*
7. CPU reads Data from BUS
8. CPU unsets Control Line */READ*

```text
                  __     __     __     __     __   
BCLK          ___|  |___|  |___|  |___|  |___|  |__
                        T1    T2            Tn
CPU WRITE                 ____________ 
A0-A15        -----------<____________>------------

CPU SET       ____________                      ___
/READ                     |____________________|

DEVICE SET    _____________                ________
/WAIT                      |______________|

CPU READ                                      _
D0-D7         -------------------------------<_>---

```


### The CPU
CPU implementation:
- The CPU instructions used to access the memory are the same used for accessing I/O devices.
- BUS content remains the same until next change.
- WAIT signal bypass the instruction cycle with NOP execution.



## Hardware

### Backplane

### Cards
* [32KB SRAM](cards/sram-32KB)



## Changes
See file [CHANGES](CHANGES.md) for the project resources change logs



## Future Plans
See file [TODO](TODO.md) for the project future plans



## About
Author : Alessandro Fraschetti (mail: [gos95@gommagomma.net](mailto:gos95@gommagomma.net))



## License
This project is licensed under the [Creative Commons BY-SA 3.0](http://creativecommons.org/licenses/by-sa/3.0/) License
