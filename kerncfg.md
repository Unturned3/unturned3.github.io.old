Absolute Minimum kernel configuration
=====================================

First, generate a minimum configuration by running:
$ make tinyconfig

Then manually tweak a few other important options:
$ make nconfig
This launches the kernel configuartion menu. Navigate around using arrow keys.
You can select/deselect options with the space bar.
NOTE: (Y) means you should select the option. (N) means you should deselect it.

64-bit kernel (Y)
General setup
	Embedded system (Y)
Enable the block layer (Y)
	Partition Types
		Advanced partition selection (Y)
			PC BIOS support (Y)
			EFI GUID Partition support (N)
Processor type and features
	Processor family (Generic-x86-64)
	Supported processor vendors (Y)
		Support Intel processors (Y)
		Support AMD processors (Y)
		Support Centaur processors (N)
	CPU microcode loading support (Y)
		Intel microcode loading support (Y)
		AMD microcode loading support (Y)
Bus options
	PCI support (Y)
		Support mmconfig PCI config space access (Y)
Executable file formats
	Kernel support for ELF binaries (Y)
	Enable core dump support (Y)
Device Drivers
	Generic Driver Options
		Support for uevent helper (Y)
		Maintain a devtmpfs filesystem to mount at /dev (Y)
			Automount devtmpfs at /dev, after the kernel mounted the rootfs (Y)
		Allow device coredump (Y)
	SCSI device support
		SCSI device support (Y)
		SCSI disk support (Y)
		SCSI CDROM support (Y)
	Serial ATA and Parallel ATA drivers (Y)
		ATA ACPI Support (Y)
		AHCI SATA support (Y)
		Intel ESB, ICH, PIIX3, PIIX4, PATA/SATA support (Y)
	Input device support
		Generic input layer (Y)
			Mouse interface (Y)
	Character devices
		Enable TTY (Y)
		/dev/mem virtual device support
		Serial drivers
			8250/16550 and compatible serial support (Y)
			Console on 8250/16550 and compatible serial port (Y)
			Serial device bus (Y)
				Serial device TTY port controller (Y)
	Graphics support
		Backlight & LCD device support (Y)
	USB support (Y)
		Support for Host-side USB (Y)
File systems
	The Extended 4 (ext4) filesystem (Y)
		Use ext4 for ext2 file systems (Y)

Exit the configuration menu by pressing F9. Save the current configuration when prompted.

Compile the kernel:
===================

$ make -jN
NOTE: replace -jN with the number of CPU cores you have on your computer. 
	  For example, on my computer I have 2 cores, so I will type "make -j2"
	  This enables make to use parallel building whenever suitable and it
	  speeds up the compilation process. 

After the compiling finished, you can find the kernel image in
/path/to/kernel-source/arch/x86/boot/
The name of the image is called "bzImage"
