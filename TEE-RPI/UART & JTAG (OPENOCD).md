# UART & JTAG (OPENOCD)
## 1. UART

### Connection in RPI
The Raspberry Pi serial port consists of two signals (a 'transmit' signal, TxD and a 'receive' signal RxD) made available on the GPIO header. To connect to another serial device, you connect the 'transmit' of one to the 'receive' of the other, and vice versa. You will also need to connect the Ground pins of the two devices together.



| UART Pin | Signal   | GPIO     |   Mode     | Header Pin |
| -------- | -------- | -------- |   -------- |--------    |
|Black (GND)     | GND     | N/A     | N/A       |6       |
|White (RXD)    | TXD    | GPIO14     | ALT0       | 8      |
|Green (TXD)    | RXD     | GPIO15     | ALT0       | 10      |

![](https://elinux.org/images/1/13/Adafruit-connection.jpg)


### Connection to PC
You can connect the Raspberry Pi to a PC using a USB-serial cable, or (if it has an RS-232 port) a level-converter circuit - see above for details. When this is done, you will need to set up a terminal emulator program on your PC as described below.
Console serial parameters
The following parameters are needed to connect to the Raspberry Pi console, and apply on both Linux and Windows.
```
Speed (baud rate): 115200
Bits: 8
Parity: None
Stop Bits: 1
Flow Control: None
```
### Linux terminal Set up
If your PC is running Linux, you will need to know the port name of its serial port. Check using following command as soon as you connect the cable:
```
$ dmesg
```
Built-in (standard) Serial Port: the Linux standard is /dev/ttyS0, /dev/ttyS1, and so on
USB Serial Port Adapter: /dev/ttyUSB0, /dev/ttyUSB1, and so on.
Some types of USB serial adapter may appear as /dev/ttyACM0 
```
ls -l /dev/ttyUSB0
```
### Minicom 
```
$ sudo apt-get install minicom
$ sudo minicom -s
```
#### Minicom Settings
```
    | A -    Serial Device      : /dev/ttyUSB0                              |
    | B - Lockfile Location     : /var/lock                                 |
    | C -   Callin Program      :                                           |
    | D -  Callout Program      :                                           |
    | E -    Bps/Par/Bits       : 115200 8N1                                |
    | F - Hardware Flow Control : No                                        |
    | G - Software Flow Control : No  
```
## 2. JTAG (OpenOCD)
### Enable UART in RPI
Enable JTAG you need to uncomment the line in rpi3/firmware/config.txt (SD Card)
```
enable_jtag_gpio=1
```
### RPI PIN Configuration

|JTAG Pin	|Signal|	GPIO	|Mode	|Header Pin|
| --------| --------| --------|--------|--------|
|1	|3v3	|N/A	|N/A	|1|
|3	|nTRST|	GPIO22|	ALT4|	15|
|5	|TDI|	GPIO26|ALT4	|3|
|7	|TMS|	GPIO27|	ALT4|	13|
|9	|TCK|	GPIO25|	ALT4|	22|
|11|	RTCK	|GPIO23	|ALT4|	16|
|13	|TDO|	GPIO24	|ALT4|	18|
|18	|GND|	N/A	|N/A|	14|
|20	|GND|	N/A	|N/A|	20|

### OpenOCD
##### Install OpenOCD
```
$ sudo apt-get install libusb-1.0-0-dev
```
##### Configure & Build
```
$ git clone http://repo.or.cz/openocd.git 
$ cd openocd
$ ./bootstrap
$ ./configure
or
$ ./configure --enable-legacy-ft2232_libftdi
$ make
$ make install
```
##### OpenOCD Configuration File - <rpi3_repo_dir>/build/rpi3/debugger/pi3.cfg
```
...
target create $_TARGETNAME_0 aarch64 -chain-position $_CHIPNAME.dap -dbgbase 0x80010000 -ctibase 0x80018000
#target create $_TARGETNAME_1 aarch64 -chain-position $_CHIPNAME.dap -dbgbase 0x80012000 -ctibase 0x80019000
#target create $_TARGETNAME_2 aarch64 -chain-position $_CHIPNAME.dap -dbgbase 0x80014000 -ctibase 0x8001a000
#target create $_TARGETNAME_3 aarch64 -chain-position $_CHIPNAME.dap -dbgbase 0x80016000 -ctibase 0x8001b000
...
```
Enable other CPU subsquently based on requirement.

##### Running OpenOCD
```
$ cd <openocd>
$ ./src/openocd -f ./tcl/interface/jlink.cfg \
-f <rpi3_repo_dir>/build/rpi3/debugger/pi3.cfg
```
File in OpenOCD - <OpenOCD>/tcl/interface/jlink.cfg 
    
##### To be able to write commands to OpenOCD, you simply open up another shell and type:
```
$ nc localhost 4444
```
## 3. Source
```
https://www.op-tee.org/docs/rpi3/
https://elinux.org/RPi_Serial_Connection
```