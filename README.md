# cross-com
A basic guide on cross compiling for ARM linux environment

## Setting up an emulated Raspberry Pi Linux environment
Credits to : http://embedonix.com/articles/linux/emulating-raspberry-pi-on-linux/

### Installing QEMU
QEMU is used to emulate the raspberry pi hardware. 
To install, use `sudo apt-get install qemu`

### Preparing linux kernel and Raspian OS image for QEMU
Download the kernel here: https://github.com/dhruvvyas90/qemu-rpi-kernel/blob/master/kernel-qemu-4.4.34-jessie?raw=true
Download the image (raspbian jessie lite) here: https://www.raspberrypi.org/downloads/raspbian/

#### Config script

Copy the command below into a text editor and name the file *config*
```
#!/bin/bash
# Starts raspberry pi image in configuration mode

qemu-system-arm -kernel ./kernel-qemu-4.4.34-jessie -cpu arm1176 -m 256 -M versatilepb -no-reboot -serial stdio -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw init=/bin/bash" -hda rpi.img
```
Make the script executable using `chmod +x config` and run it using `./config`. This launches QEMU raspberry pi in bash terminal mode.

Now in QEMU, use `nano /etc/ld.so.preload` to edit the file and comment out the only line using #. (i.e *#/usr/lib/arm-linux-gnueabihf/libarmmem.so*) Use CTRL+O and Enter to save the changes to the file and CTRL+X to exit.

Since the actual Pi system uses SD Card but QEMU does not have that (we defined the root drive to be sda2 as seen in the config script), we have to redirect calls to the SD Card to sda2. To do that we need to define rules for the symbolic link.

In QEMU, use `nano /etc/udev/rules.d/90-qemu.rules` to create a new rule file. In the nano editor, type in the following:

```
KERNEL=="sda", SYMLINK+="mmcblk0"
KERNEL=="sda?", SYMLINK+="mmcblk0p%n"
KERNEL=="sda2", SYMLINK+="root"
```

#### Run script

Use the following script to launch QEMU pi with raspbian OS
```
#!/bin/bash
# Start the Raspberry Pi in fully functional mode!
 
qemu-system-arm -kernel ./qemu-rpi-kernel/kernel-qemu -cpu arm1176 -m 256 -M versatilepb -no-reboot -serial stdio -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" -hda rpi.img
```

#### Login Defaults
Username: pi <br>
Password: raspberry


## Compiling C programs with ARM linux toolchain

## Transferring C program to Raspberry Pi using SCP
