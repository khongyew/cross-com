# cross-com
A basic guide on cross compiling on an Ubuntu PC for an ARM linux environment <br>
Mainly to document what I have learnt from the web. I do not claim to be an expert.

## Setting up an emulated Raspberry Pi Linux environment
Reference : http://embedonix.com/articles/linux/emulating-raspberry-pi-on-linux/

### Installing QEMU
QEMU is used to emulate the raspberry pi hardware. 
To install, use `sudo apt-get install qemu`

### Preparing linux kernel and Raspian OS image for QEMU
Download the kernel here: https://github.com/dhruvvyas90/qemu-rpi-kernel/blob/master/kernel-qemu-4.4.34-jessie?raw=true <br>
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


## Compiling C programs with ARM linux toolchain. (Hello world example)

Reference: http://gnutoolchains.com/raspberry/tutorial/

Install the GCC compiler toolchain for ARM linux using `sudo apt-get install gcc-6-arm-linux-gnueabi`

Save the following code as *helloworld.c*

```
#include <stdio.h>

int main()
{
    printf("Hello world!\n");
    return 0;
}
```

Compile the code using `arm-linux-gnueabi-gcc-6 helloworld.c -o test`

## Transferring C program to Raspberry Pi using SCP

Reference: https://www.youtube.com/watch?v=Hj7UWfhv7Gc

Now we need to transfer this *test* file to the QEMU pi. But first we need to make changes to the run script

### Port forwarding
```
#!/bin/bash
# Start the Raspberry Pi in fully functional mode!

qemu-system-arm -kernel ./kernel-qemu-4.4.34-jessie -cpu arm1176 -m 256 -M versatilepb -no-reboot -serial stdio -append "root=/dev/sda2 panic=1 rootfstype=ext4 rw" -redir tcp:2222::22 -hda rpi.img
```

The difference between this and the old script is additional the `-redir tcp:2222::22`, which fowards port 2222 of the host machine to port 22 of the virtual machine (QEMU pi). This allows us to call port 2222 (using SCP) on the local machine to make transfers to the QEMU pi.

### Enabling SSH in QEMU pi

Use `sudo raspi-config` to bring up the raspberry configuration menu

Follow the screenshots to enable SSH:

![Image 1](/images/interfacing_options.png)

![Image 2](/images/ssh.png)

![Image 3](/images/select_yes.png)

![Image 4](/images/final_screen.png)

Press *ESC* to exit the config menu

### Transferring using SCP

The following command transfers the *test* file to the */home/test* directory in pi
```
scp -P 2222 test pi@localhost:/home/pi/test
```
Inside QEMU pi *test* directory, use `chmod +x test` to make the *test* file executable. Run it using `./test` to see *Hello world!* printed

