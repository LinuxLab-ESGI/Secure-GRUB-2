<link href="/style/style.css" rel="stylesheet"></link>

# Securing-GRUB-2

This is a tutorial which explain how to secure the access of GRUB 2 at Linux startup.

Table of Content

- [Securing-GRUB-2](#securing-grub-2)
  - [GRUB](#grub)
  - [Boot process of Linux](#boot-process-of-linux)
    - [1. BIOS](#1-bios)
    - [2. MBR](#2-mbr)
    - [3. GRUB](#3-grub)
    - [4. Kernel](#4-kernel)
    - [5. Init](#5-init)
  - [The "flaw" of GRUB](#the-flaw-of-grub)
    - [Access to a shell with root privileges](#access-to-a-shell-with-root-privileges)
  - [Secure GRUB with an encrypted password](#secure-grub-with-an-encrypted-password)
  - [Allow automatic boot](#allow-automatic-boot)

## GRUB

First of all GNU GRUB (**GR**and **U**nified **B**ootloader) is a boot loader and boot manager from the [GNU Project](https://www.gnu.org/).  
Excuted after the booting of the BIOS or UEFI it allows user to select in which operating system he wants to boot and it serves in the same time as a Linux boot loader.

## Boot process of Linux

They are usually 5 different stages of Linux boot process involved during the startup of a Linux distro :

### 1. BIOS

The BIOS (**B**asic **I**nput **O**utput **S**ystem) is stored in a memory location of the motherboard called EEPROM (**E**lectrically-**E**rasable **P**rogrammable **R**ead-only **M**emory), the BIOS is a set of functions (written in Assembly Language) which allow the first interaction with the physical material (Hard drives, Keyboard, Mouse, ...).  
It scans all the internal and external devices and interfaces connected to the motherboard and performs some system integrity checks.  
Then, depending of the boot order configured in the BIOS, it loads and executes the 512 bytes of the disk, know as the MBR.

> Nowadays the BIOS is replaced by UEFI (**U**nified **E**xtensible **F**irmware **I**nterface) that can act as a boot loader and manager and a replacement of GRUB.

### 2. MBR

The first 512 bytes of the disk are called the MBR (**M**aster **B**oot **R**ecord). It conatains :

- Boot code or Bootstrap code which contains informations about the boot loader (446 bytes)
- Partitions table to indesx all the partitions of the disk (64 bytes)
- Boot signature to check if the disk is bootable or not (2 bytes)

In a nutshell, the MBR is charged of loading and executing the boot loader in our case : GRUB.

### 3. GRUB

Here comes our famous GRUB ! :smile:  
It loads all the available operating system (Linux kernels precisly) or other boot loaders like Windows Boot Manager.
If we don't do anything at the GRUB screen, it loads and executes automatically the default Linux kernel (vmlinuz) and initrd (inital ramdisk) images.

> Loaded into the RAM, initrd is a filsystem which is used for the Linux startup process. It contains all the additional modules and drivers for the the kernel.

### 4. Kernel

Mounts the root file system as specified in the “root=” in grub.conf
Kernel executes the /sbin/init program
Since init was the 1st program to be executed by Linux Kernel, it has the process id (PID) of 1. Do a ‘ps -ef | grep init’ and check the pid.
initrd is used by kernel as temporary root file system until kernel is booted and the real root file system is mounted. It also contains necessary drivers compiled inside, which helps it to access the hard drive partitions, and other hardware.

### 5. Init

Looks at the /etc/inittab file to decide the Linux run level.
Following are the available run levels
0 – halt
1 – Single user mode
2 – Multiuser, without NFS
3 – Full multiuser mode
4 – unused
5 – X11
6 – reboot
Init identifies the default initlevel from /etc/inittab and uses that to load all appropriate program.
Execute ‘grep initdefault /etc/inittab’ on your system to identify the default run level
If you want to get into trouble, you can set the default run level to 0 or 6. Since you know what 0 and 6 means, probably you might not do that.
Typically you would set the default run level to either 3 or 5.

## The "flaw" of GRUB

As we mentionned earlier, the kernel executes normally the **/sbin/init** program. However we can shortcut this procedure by executing another program instead. For instance we can excute a shell and simply have an acess to the machine with root privileges additionaly. :confused:  

### Access to a shell with root privileges

Here we are going to see how to have acess to the machine whitout typing any login or password.  

>We are working with a simple default Debian 10 with a user account in it.

First of all we boot the machine and arrive at the GRUB boot menu :

![Boot Menu Image](/img/BootMenu.png#center "A very nice and simple Boot Menu")

At this stage press "**e**" to edit the boot option of the kernel :

![Edit Boot Menu Image](/img/EditBootMenu.png#center "Here we can't choose our favorite text editor xD")

Now we go to the line (near the bottom) that contains "**/boot/vmlinuz**..."

![Vmliuz line Image](/img/Vmlinuz.png#center "Here is our root for the filesystem ^^")

We just replace "**ro**" (read only) by "**rw**" (read and write) in order to write in the disk and add the init option with the location of shell in argument "**init=/bin/bash**".

Here are telling to the kernel to execute the bash shell (the most commun shell, usually by default) instead of the init program (/sbin/init)

Here is the line once modified :

![Vmlinuz modified Image](/img/Vmlinuz2.png#center "Just simple as that -.-")

Now we simply press "**F10**" to boot with the option we've added.

Since the init process is executed with root privileges ... Tada :boom: we have access to the machine with the user root ! You could do anaything you want at this stage :

![Root access Image](/img/Root.png#center "With great power comes great responsibility")

That's why it is very important to secure our GRUB.

## Secure GRUB with an encrypted password

:warning: If someone has physical access to your machine and is able to boot a live USB/disk (i.e., BIOS allows booting from an external disk), it is fairly trivial for one to modify GRUB configuration files to bypass this if /boot resides on an unencrypted partition

## Allow automatic boot

___
Author : AnthonyF
