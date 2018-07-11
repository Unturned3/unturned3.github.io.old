# MicroLinux =================================================================

	The MicroLinux project will guide you through the process of building a 
functional yet compact linux system from scratch. The end result is a bootable
disk image that contains a custom linux kernel, musl libc, a service manager,
a lightweight shell, and ~150 common system utilites in around 6MB of space.

# Why? =======================================================================

	Originally MicroLinux started out as a personal project to develop a tiny
linux system that uses as little disk space and RAM as possible (intended as
a Xen hypervisor guest). However during my research I found out that most of the
information is badly outdated. A lot of the articles/documentation I found was
written way back in 2005 and as a result the instructions were not applicable
any more. Also, there are little detailed information regarding the subject
of configuring the Linux kernel, which is a vital part of the system. Because
of this I basically built my system using trail-and-error. The process was 
extremely tedious, as it involved repetitive tweaking of hundreds of switches
in the configuration files. One wrong option could potentially render your
system unbootable (maybe due to missing hardware drivers, etc.), or give out
confusing errors at runtime.

