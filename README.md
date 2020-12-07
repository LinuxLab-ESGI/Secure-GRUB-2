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

The Linux Kernel first mounts the root file system set in grub.conf in the line **root=**.  
Then it executes the **/sbin/init** program as the fisrt program with root privileges which executes some others scripts.

> **/sbin/init** is actually a symbolic to the init system of the OS. It is either SysV or Systemd. Nowadays systemd is the most used  and it is compatible with SysV init scripts  
> If it is SysV all the scripts and programs are located in **/etc/systemd/system/** and **/lib/systemd/system/**  
> So **/sbin/init** is a symbolic link to **/lib/systemd/systemd**

Since init is the first program to be executed by Linux kernel, it has the PID (**Process** **ID**entifier) of 1.  
We can do a ps command to check the first PID :  
`ps -ef | grep init`

The kernel then establishes a temporary root file system with initrd until the real file system is mounted. It also contains necessary drivers compiled inside as we saw earlier.

### 5. Init

First the init program reads its initialization files which are in **/etc/init.d/** (/etc/inittab before with SysV). It sets everthing  the system needs for its initialization.  
Then it set the default run level. A run level is a configuration of processes.

There are 7 run level from 0 to 1 :

- 0 – Halt
- 1 – Single user mode
- 2 – Multi-user
- 3 – Full multi-user mode
- 4 – Unused
- 5 – X11 (Full multi-user graphical mode)
- 6 – Reboot

> To check the current run level you can do :  
> `who -r`  
> `sudo runlevel`  
> `systemctl get-default` *for systemd*  
> It should normally output the run level 5 or 3

Modern Linux systems use systemmd which refers with this :

- 0 - poweroff.target
- 1 - rescue.target
- 2,3,4 - multi-user.target
- 5 - graphical.target
- 6 - reboot.target

> To set a run level with systemd you can do this command :  
> `sudo systemctl isolate name.target`  

## The "flaw" of GRUB

As we mentionned earlier, the kernel executes normally the **/sbin/init** program. However we can shortcut this procedure by executing another program instead. For instance we can excute a shell and simply have an acess to the machine with root privileges additionaly. :confused:  

### Access to a shell with root privileges

Here we are going to see how to have acess to the machine whitout typing any login or password.  

> We are working with a simple default Debian 10 with a user account in it.

First of all we boot the machine and arrive at the GRUB boot menu :

![Boot Menu Image](/img/BootMenu.png "A very nice and simple Boot Menu")

At this stage press "**e**" to edit the boot option of the kernel :

![Edit Boot Menu Image](/img/EditBootMenu.png "Here we can't choose our favorite text editor xD")

Now we go to the line (near the bottom) that contains "**/boot/vmlinuz**..."

![Vmliuz line Image](/img/Vmlinuz.png "Here is our root for the file system ^^")

We just replace "**ro**" (read only) by "**rw**" (read and write) in order to write in the disk and add the init option with the location of shell in argument "**init=/bin/bash**".

Here are telling to the kernel to execute the bash shell (the most commun shell, usually by default) instead of the init program (/sbin/init)

Here is the line once modified :

![Vmlinuz modified Image](/img/Vmlinuz2.png "Just simple as that -.-")

> Here are some options that we can give :  
>
> - root=device : specifies the disk where we want to mount the root file system (Exemple : /dev/sdaX, LABEL, UUID
> - init=binary : specifies the initial program exceuted by the kernel (Normally /sbin/init)  
> - single : allow to start in single user (root) with minimum services  
> - ro : the file stystem will be mounted in read-only and do a file system consistency check (fsck)
> - rw : the file stystem will be mounted in read and write and doesn't do a fsck.
> - quiet : dont' user verbose mode.

Now we simply press "**F10**" to boot with the option we've added.

Since the init process is executed with root privileges ... Tada :boom: we have access to the machine with the user root ! You could do anaything you want at this stage :

![Root access Image](/img/Root.png "With great power comes great responsibility")

That's why it is very important to secure our GRUB.

## Secure GRUB with an encrypted password

What we are actually doing is to add a login/password to GRUB configuration's files whenever the user wants to edit the boot option.

:warning:  
If we can boot a live USB/disk on our machine and the partition **/boot** is not encrypted we can easily modify the GRUB configuration to bypass what we did before.  
So to really secure our GRUB we have to make sure that the **/boot** partition is encrypted.

## Allow automatic boot

___
Author : AnthonyF
