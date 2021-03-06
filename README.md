# avnav-seatalk1-simulator-rs232

![grafik](https://user-images.githubusercontent.com/98450191/153687947-2c0b2547-b184-4cfd-bf48-38300e376872.png)

# General

The plugin simulates seatalk 1 protocol cyclic outputs via RS232.
It sets the depth (DBT) and speed (STW) like ST50Plus Tridata does.

It is widely based on the seatalk remote plugin (https://github.com/wellenvogel/avnav-seatalk-remote-plugin).

# Parameter

- device: e.g. '/dev/ttyUSB0'
- usbid: as alternative for devive name

# Details

The Seatalk 1 protocol is simply written cyclic to the selected device.
The following bytes are written to RS232:

DPT 'Depth below transducer' of 6,7 meters
- Byte 1: Command byte             : 0x00 => parity bit is set to mark up the start of Seatalk protocol
- Byte 2: Attribute byte           : 0x02 => LSB is the number of following bytes (n=2)
- Byte 3: First mandatory data byte: 0x00 => YZ: in feets
- Byte 4: optionially data byte 1  : 0xdd: LSB for value for 'Depth below transducer'
- Byte 5: optionially data byte 2  : 0x00: MSB for value for 'Depth below transducer'
The resulting value for 'Depth below transducer' is 0x00dd/10 feets (221/10 feets = 22,1 feets = 6,7 meters).

STW 'Speed Through Water' of 10,9 km/h
- Byte 1: Command byte             : 0x20 => parity bit is set to mark up the start of Seatalk protocol
- Byte 2: Attribute byte           : 0x01 => LSB is the number of following bytes (n=1)
- Byte 3: First mandatory data byte: 0x3B => LSB for value for 'Speed Through Water'
- Byte 4: optionially data byte 1  : 0x00: MSB for value for 'Speed Through Water'
The resulting value for 'Speed Through Water' is 0x003b/10 kn (59/10 kn = 5,9 kn = 10,9 km/h).

It is possible that these plugin and avnav-anchor-chain-simulator-rs232 plugin can share the same device (e.g. "/dev/ttyUSB_SeatalkOut")

# Hardware needs
- A second USB-RS232-adapater
- A gender changer between USB-RS232-adapater of plugin avnav-seatalk1-reader-rs232 and the adapter of these plugin

# Installation
To install this plugin please 
- install packages via: sudo apt-get update && sudo apt-get install python3-serial
- create directory '/usr/lib/avnav/plugins/seatalk1-simulator-rs232' and 
- copy the file plugin.py to this directory.

# TODOs
- only tested with linux

# Helpers
Setup the serial devices by their serial numbers
- Label your first USB serial device (e.g SeatalkOut)
- Connect the first USB serial device to the PC
- Get the vendorID, deviceID and serial number of the tty device (here "/dev/ttyUSB0")
   udevadm info -a -n /dev/ttyUSB0 | grep {idVendor} | head -n1  => ATTRS{idVendor}=="0403" 
   udevadm info -a -n /dev/ttyUSB0 | grep {bcdDevice} | head -n1 => ATTRS{bcdDevice}=="0600"
   udevadm info -a -n /dev/ttyUSB0 | grep {serial} | head -n1    => ATTRS{serial}=="A10KKBM3"
- creates an udev rule
  mcedit sudo mcedit /etc/udev/rules.d/10-local.rules
   SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="A10KKBM3", MODE="0666", SYMLINK+="ttyUSB_SeatalkOut"
- Continue with the next devices
- at the end the file /etc/udev/rules.d/10-local.rules may look like that
    SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="A10KKF9V", MODE="0666", SYMLINK+="ttyUSB_SeatalkInp"
    SUBSYSTEM=="tty", ATTRS{idVendor}=="0403", ATTRS{idProduct}=="6001", ATTRS{serial}=="A10KKBM3", MODE="0666", SYMLINK+="ttyUSB_SeatalkOut"
- Use this names in avnav (e.g: "/dev/ttyUSB_SeatalkOut")
