# AMICO - Another Micro 8-bit Computer
This is a home project for a simple 8-bit microcomputer, based on a CPU implemented by a microcontroller (as a PIC or similar).

![amico-mercury.jpg](images/amico-mercury.jpg)



## Features
- 8-bit Data BUS
- 16-bit Address BUS
- Memory mapped I/O
- Programmable OSC/n signal
- Reset control signal
- Wait control signal for slow device operations


## Specifications

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
    * /WAIT - wait for device operation. "Lo" value puts the CPU into IDLE mode until line come back "Hi".
* System lines :
    * /RESET - system reset. "Lo" value causes a system reset.
    * OSC - system clock. 
    * OSC/n - programmable clock divider.


### Address Map
The memory and registers of the I/O devices are mapped on the same address space (memory mapped I/O), 
so a memory address may refer to either a portion of physical RAM/ROM or to registers of the I/O device.
The address map of the AMICO system looks like this:

| Addresses     | Size | Assignment   |
|---------------|------|--------------|
| C000H - FFFFH | 16KB | USER ROM 0,1 |
| A000H - BFFFH |  8KB | SYSTEM ROM   |
| 8000H - 9FFFH |  8KB | I/O DEVICE   |
| 0000H - 7FFFH | 32KB | RAM 0,1,2,3  |


### Basic System Operations
System operations are signal based sequence of commands, directed from CPU to DEVICES.


#### Write Operation
1. CPU writes and latches Address on BUS
2. CPU sets Control Line */WRITE*
3. CPU writes Data on BUS
4. CPU unsets Control Line */WRITE*
5. DEVICE latches Data from BUS

#### Read Operation
1. CPU writes and latches Address on BUS
2. CPU sets Control Line */READ*
3. DEVICE writes Data on BUS
4. CPU reads Data from BUS
5. CPU unsets Control Line */READ*
6. DEVICE releases BUS

#### Slow Device I/O Write Operation
1. CPU writes and latches Address on BUS
2. CPU sets Control Line */WRITE*
3. CPU writes Data on BUS
4. CPU unsets Control Line */WRITE*
5. DEVICE sets Control Line */WAIT*
6. CPU enters into IDLE mode
7. DEVICE latches Data from BUS
8. DEVICE unsets Control Line */WAIT*

#### Read Operation
1. CPU writes and latches Address on BUS
2. CPU sets Control Line */READ*
3. DEVICE sets Control Line */WAIT*
4. CPU enters into IDLE mode
5. DEVICE writes Data on BUS
6. DEVICE unsets Control Line */WAIT*
7. CPU reads Data from BUS
8. CPU unsets Control Line */READ*
9. DEVICE releases BUS


### The CPU
CPU implementation:
- The CPU instructions used to access the memory are the same used for accessing I/O devices.
- BUS content remains the same until next change.
- WAIT signal bypass the instruction cycle with NOP execution.



## Changes
See file [CHANGES](CHANGES.md) for the project resources change logs



## Future Plans
See file [TODO](TODO.md) for the project future plans



## About
Author : Alessandro Fraschetti (mail: [gos95@gommagomma.net](mailto:gos95@gommagomma.net))



## License
This project is licensed under the [Creative Commons BY-SA 3.0](http://creativecommons.org/licenses/by-sa/3.0/) License