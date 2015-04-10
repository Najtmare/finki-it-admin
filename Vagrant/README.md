#ArchLinux installation & Vagrant usage

##Content
- [Installing ArchLinux](#installing-archlinux)
- [Vagrant usage](#vagrant-usage)

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

###<a id="vagrant-usage"></a>Vagrant usage

First, you need to create **Vagrantfile** which will be the configuration file of the *Vagrant box* you are going to create.

	vagrant init

This command will create the **Vagrantfile_base** which we need to customize (the file will be created in the folder we are currently in)

Than, you can create the *Vagrant box* from any VM software you have installed on the computer. In command prompt (under Windows hosts), locate your self in the folder you want to have the box created (preferably the folder where you have created the **Vagrantfile**) and run the command below.

	vagrant package --base VM_NAME --output VMBoxName.box

With **--base** you specify the VM you want to create as **Vagrant box** and the **--output** specifies the *box* name.

After the box is created, you need to configure the **Vagrantfile** so it matches the proper **Vagrant box** name, configure RAM usage when deploying the box on new host, maybe set static IP addresses, SSH credentials and so on (check **Vagrantfile_configured**).

Last step is to deploy the **Vagrant box**.

	vagrant up

This will start deploying the VM from the box with the specifications given in the **Vagrantfile_configured**.