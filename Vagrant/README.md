#ArchLinux installation & Vagrant usage

##Content
- [Installing ArchLinux](#installing-archlinux)
- Vagrant usage

###<a id="installing-archlinux"></a>Installing ArchLinux

Identify the devices where the **ArchLinux** will be installed.

    lsblk

I will use **parted** to make disk partitions and install the OS. Open the device where you want to create partition table

    parted /dev/sda

and create new **msdos** partition table.

    mklabel msdos

Next step is to allocate disk space as we want, for example for the *master boot* I will allocate *100MB*.

    mkpart primary ext3 1M 100M
	set 1 boot on

Now, allocate the space that is left on the hard disk.

	mkpart primary ext3 100M 8G

Next step is to create the file system and mount the partitions.

	mkfs.ext4 /dev/sda1
	mkfs.ext4 /dev/sda2
	mount /dev/sda1 /boot
	mount /dev/sda2 /home

After mounting partitions is done, we need to install the base system and generate a *fstab*.

	pacstrap -h /home base base-devel
	genfstab -U -p /home >> /home/etc/fstab

*chroot* into the installed system

	arch-chroot /home /bin/bash

so you can configure primary files of the **ArchLinux** system. Define system language with

	nano /etc/locale.gen

and change timezone (I will configure my time zone here).

	ln -s /usr/share/zoneinfo/Europe/Belgrade /etc/localtime

Set hardware clock to UTC

	hwclock --systohc --utc

and set *root* password.

	passwd

Next, install *GRUB boot loader*

	pacman -S grub
	grub-install --target=i386-pc --recheck /dev/sda

and *OS-prober*.

	pacman -S os-prober
	grub-mkconfig -o /boot/grub/grub.cfg

Exit from **chroot**, umount the *root* and reboot.

	exit
	umount -R /mnt
	reboot