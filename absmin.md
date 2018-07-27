Typography:
'$' means a shell command executed as a normal user
'#' means a shell command executed with root privilege 
'//' means a comment, intending to explain certain shell commands


Overview
========

In this section, we will be building a _minimal_ Linux system that can run inside Quick Emulator
(QEMU). In order for the system to be called Linux, it needs to run the Linux kernel. The kernel
is a core component of the system that interacts with the hardware and manages the resources 
alocated to processes running on the computer. Without the kernel userspace applications would
need to directly interact with the hardware they are running on, which is a difficult and messy
task.

Linux Internals
===============

Generic Boot Process: 
1. hardware powers on

2. BIOS (basic input-output system) gets loaded into RAM from the on-board 
   flash storage (located somewhere on the motherboard of the computer)

3. BIOS performs hardware integrity checks & looks for the first bootable
   device it finds (CD-ROM, USB, hard drive, etc.)

4. BIOS loads the primary stage bootloader on the first bootable device.
   In case of a hard drive, the primary stage bootloader is usually located
   in the Master Boot Record, the 1st sector of the disk. The MBR is only
   512 bytes in size (446 bytes for the primary bootloaer, 64 bytes for
   storing the partition table). The job of this bootloader is to load the
   second stage bootloader. 

5. The primary bootloader loads the secondary bootloader, which is usually
   GRUB or Syslinux. These bootloaders provide you with the "boot menu"
   screen that you usually see when your computer turns on. A third bootloader
   are sometimes loaded (chain loading) but this setup is typically not needed

6. GRUB then loads the kernel image (vmlinuz/bzImage) into memory and then
   passes the control over to the running kernel. 

7. The kernel mounts the root file system, which could be an actual hard
   disk or it can be a virtual RAM file system. The kernel then executes a
   userspace program to do initialization tasks, which is usually /sbin/init,
   but there is no restriction over what can the kernel execute (can be any
   custom executable binaries / scripts)

8. /sbin/init is the first userspace process that the kernel launches and its
   PID is always 1. This process cannot exit during the runtime of the system
   or else you will end up with a kernel panic (when the kernel runs into an
   unrecoverable error and has to halt to prevent further damage).

   All other userspace processes are descendants of init, and all orphaned 
   processes (processes that has their parents killed) are automatically adopted
   by init.

   Commonly, init reads /etc/inittab in order to carry out specific initialization
   tasks, such as loading modules, mounting disks, running other scripts, starting
   daemon processes, etc. 

9. After all this, init executes /sbin/getty to open & setup a few TTYs
   (TTY: TeleTYper, the old text consoles used back in the dawn of UNIX.
   You can think of this as the terminal). Then /sbin/getty invokes login,
   which asks for the username & password on the TTY.

10. Now the user can log in and use the system.

Since this is probably your first time building a Linux system from scratch, we will
omit a few steps mentioned above in order to simplify the process, namely:

- QEMU will be used to emulate the system instead of using a bootloader to run it
  on an actual machine.
- /sbin/init will be replaced by a very simple C program to print some text
- init will not launch any other userspace processes, for the sake of simplicity

Components needed for the minimal system:
- A working Linux kernel image
- A disk image for the kernel to mount as the root file system
- A init program to show that the kernel passed the control over to userpace.

Now, our modified/simplified version of the boot process looks like this:
1. QEMU loads the kernel image into RAM (replaces bootloader)
2. QEMU passes the disk image we specified to the loaded kernel
3. kernel mounts the disk image as the rootfs
4. kernel executes the custom init program, which prints some text to the screen


Constructing the System
=======================

Visite kernel.org and click on the yellow button saying "Latest Stable Kernel" to download the
Linux kernel source code. Then create a working directory in your home folder named "build" and 
copy the downloaded tarball into the build folder. Rename the tarball "kernel.tar.xz" and untar
it:

$ xz -d kernel.tar.xz
$ tar -xf kernel.tar
$ rm kernel.tar		// remove the tarball

Rename the decompressed folder to "kernel" and change directory into it.
$ cd kernel
$ make mrproper		// cleans the folder of any left over files



Absolute Minimum kernel configuration
=====================================

First, generate a minimum configuration by running:

$ make tinyconfig

Then manually tweak for a few other important options by:

$ make nconfig

This launches the kernel configuartion menu. Navigate around using arrow keys.
You can select/deselect options with the space bar.
NOTE: (Y) means you should select the option. (N) means you should deselect it.

64-bit kernel (Y)	// we are running on a 64 bit system
General setup
	Configure standard kernel features (expert users)
		Enable support for printk (Y)
	Embedded system (Y)		// reveals certain expert options
	Support for paging of anonymous memory (N)	// we will not be using swap space
Enable the block layer (Y)	// required to access hard drives
	Partition Types
		Advanced partition selection (Y)
			PC BIOS support (Y)		// partition scheme used for our disk
			EFI GUID Partition support (N)	// no need for UEFI or GPT
Processor type and features
	Processor family (Generic-x86-64)	// running on 64 bit machine
	Supported processor vendors (Y)
		Support Intel processors (Y)
		Support AMD processors (Y)
		Support Centaur processors (N)	// no need for this processor type
	CPU microcode loading support (Y)	// corrects CPU's faulty behavior
		Intel microcode loading support (Y)
		AMD microcode loading support (Y)
Bus options
	PCI support (Y)		// lots of devices connected via PCI bus
		Support mmconfig PCI config space access (Y)
Executable file formats
	Kernel support for ELF binaries (Y)		// compiling an ELF binary
	Enable core dump support (Y)	// debug purposes
Device Drivers
	Generic Driver Options
		Support for uevent helper (Y)	// device hotplugging management
		Maintain a devtmpfs filesystem to mount at /dev (Y)		// dev filesystem
			Automount devtmpfs at /dev, after the kernel mounted the rootfs (Y)
		Allow device coredump (Y)	// debug purposes
	SCSI device support		// generic hard drive support (SCSI layer)
		SCSI device support (Y)
		SCSI disk support (Y)
		SCSI CDROM support (Y)
	Serial ATA and Parallel ATA drivers (Y)	// SATA hard drive support
		ATA ACPI Support (Y)
		AHCI SATA support (Y)
		Intel ESB, ICH, PIIX3, PIIX4, PATA/SATA support (Y)
	Input device support	// drivers for keyboards
		Generic input layer (Y)
			Mouse interface (N)		// no need for mouse (keyboard only)
	Character devices
		Enable TTY (Y)	// so we can see all the printed messages & interact with the system
			Virtual terminal (N)
			Unix 98 PTY support (N)
			Legacy (BSD) PTY support (N)
		/dev/mem virtual device support (N)
		Serial drivers
			8250/16550 and compatible serial support (Y)	// generic serial drivers
			Console on 8250/16550 and compatible serial port (Y)	// serial console
			Serial device bus (Y)
				Serial device TTY port controller (Y)
		/dev/port character device (N)
	USB support (Y)		// support for USB devices (such as keyboards, etc.)
		Support for Host-side USB (Y)
File systems
	The Extended 4 (ext4) filesystem (Y)	// filesystem for our disk image
		Use ext4 for ext2 file systems (Y)

Exit the configuration menu by pressing F9. Save the current configuration when prompted.


Compiling the kernel:
=====================

$ make -jN
NOTE: replace -jN with the number of CPU cores you have on your computer. 
	  For example, on my computer I have 2 cores, so I will type "make -j2"
	  This enables make to use parallel building whenever suitable and it
	  speeds up the compilation process. 

After the compiling finished, you can find the kernel image in
~/build/kernel/arch/x86/boot/
The name of the image is called "bzImage"

Creating the Disk Image
=======================

Create a workin directory inside ~/build named "miniSys" and copy the kernel image over

$ mkdir ~/build/miniSys
$ cp ~/build/kernel/arch/x86/boot/bzImage ~/build/miniSys/bzImage
$ cd ~/build/miniSys

Construct a disk image for the minimum system

$ touch mini.img
$ dd if=/dev/zero of=./mini.img bs=1M count=2	// create a 2MB disk image
$ mkdir mnt		// temporary mount point
# losetup -Pf --show mini.img

NOTE: the losetup command will return something like "/dev/loop0". However on your
system the loop device might already be occupied, so it might be something like
"/dev/loop1" or "/dev/loop2". We will use "/dev/loopN" to represent the device.
Replace "N" with the number on your system.

Use the fdisk utility to partition the disk image (attached to /dev/loop0)
# fdisk /dev/loopN

o: new BIOS (aka DOS) partition table
n: create new partition
p: make this new partition as primary
1: make this the 1st partition
1: start from sector 1
+4094: let this partition fully occupy the 2MB disk image
a: toggle the bootable flag for the partition
w: write the new partition to /dev/loop0
q: quit fdisk

Now you can see a partition named /dev/loopNp1. Create an EXT4 file system on the partition:
NOTE: "p1" means the first partition on the device "/dev/loopN" (we only created 1 partition)

# mkfs.ext4 /dev/loopNp1
# mount /dev/loopNp1 mnt


Creating the Custom Init Program
================================

We are going to write a tiny program using the C programming language. Don't worry if you have
no experience in programming.

$ touch init.c
$ vim init.c	// you can use any text editor to edit the file (I'm using VIM)

Now, type the following text into init.c 

#include <stdio.h>
void main()
{
	printf("\n\nHello, MicroLinux!\n\n");
}

* The "printf" statement prints the text "Hello, MicroLinux" to the screen.
* The "#include <stdio.h>" statement tells the compiler to look into a file named "stdio.h",
  because printf is written in that file.
* "void main()" is the entry point of the program (the start of execution)
* The curly braces defines what "main" will be containing, in this case, the printf statement

Save the file and exit the editor. Compile the program:

$ gcc init.c -o init --static

The "--static" option tells the compiler to compile init.c as a _static_ executable. When
an executable is static, it means that it has no dependency on any external libraries or files.
We need our init program to be static since adding external libraries complicates things.

You should now see a program named "init" in the current directory. You can run it to check
that everything is working correctly.


Assemblying the System
======================

Copy our custom init program into the disk image mount:

# cp init ./mnt/init
# umount ./mnt


Testing!
========

Run the system using QEMU:

$ qemu-system-x86_64 -kernel ./bzImage -append "root=/dev/sda1 rootfstype=ext4 \
  init=/init console=ttyS0,9600" -nographic mini.img

Meaning of options:
-kernel: this specifies the kernel image we would like to load into RAM
-append: parameters supplied into the kernel.
	* root=/dev/sda1
	  This tells the kernel what to mount as the rootfs. In this case, /dev/sda1
	  is our disk image (loaded by QEMU) containing the init program
	* rootfstype=ext4
	  This tells the kernel what file system is on the device sda1
	* init=/init
	  This tells the kernel to execute /init as the init program
	* console=ttyS0,9600
	  This tells the kernel to print all its text messages to serial port 0 (ttyS0)
	  9600 specifies the baud rate to use for the serial port
-nographic: tells QEMU to use a serial port instead of a virtual terminal
mini.img: tells QEMU which disk image to load
