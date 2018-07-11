# MicroLinux =================================================================

	The MicroLinux project will guide you through the process of building a 
functional yet compact linux system from scratch. The end result is a bootable
disk image that contains a custom linux kernel, musl libc, a service manager,
a lightweight shell, and ~150 common system utilites in around 6MB of space.

# Why? =======================================================================

	Originally MicroLinux started out as a personal project to develop a tiny
linux system that uses as little disk space and RAM as possible (intended as
a Xen hypervisor guest). However most of the articles/documentation out there
is badly outdated (written way back in 2005) and sometimes there was no docs
to read at all (such as how to configure a linux kernel). I had to build the
system using trial-and-error, and as a result I learned quite a lot about the
internals of linux. The entire process was very tedious and difficult, so I
thought it would be good to share my experience.

