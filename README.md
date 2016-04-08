# Server
These scripts set up a server with ArchLinux and Btrfs

## Arch Linux
Download the latest iso from `https://www.archlinux.org/download`, and load it onto a bootable USB drive

Insert the USB drive into the machine, turn it on, and select the USB drive as the boot device

### Internet
Verify that the internet works by running

	ping -c 4 google.com

If there are problems, go [here](https://wiki.archlinux.org/index.php/Network_configuration)

### System Clock

    timedatectl set-ntp true

Verify everything worked with

	timedatectl status

### Btrfs
Assuming drives `/dev/sd[a-d]`

    mkfs.btrfs -L main -m raid10 -d raid10 /dev/sda /dev/sdb /dev/sdc /dev/sdd

	mount -t btrfs /dev/sda /mnt

    btrfs subvolume create /mnt/@        # root
	btrfs subvolume create /mnt/@boot    # boot
	btrfs subvolume create /mnt/@home    # home

	umount /mnt

	mount -t btrfs -o noatime,autodefrag,compress-force=lzo,space_cache,subvol=@ /dev/sda /mnt

	mkdir /mnt/boot
	mount -t btrfs -o noatime,autodefrag,compress-force=lzo,space_cache,subvol=@boot /dev/sda /mnt/boot

	mkdir /mnt/home
	mount -t btrfs -o noatime,autodefrag,compress-force=lzo,space_cache,subvol=@home /dev/sda /mnt/home

### Install Packages
Update `/etc/pacman.d/mirrorlist` by using [Pacman Mirrorlist Generator](https://www.archlinux.org/mirrorlist/)

Afterwards, install the `base` and `base-devel` packages

    pacstrap -i /mnt base base-devel

This part will take 5-10 minutes

### Fstab

    genfstab -L /mnt >> /mnt/etc/fstab

Modify `/mnt/etc/fstab` and simplify subvolume references. For example, replace `subvolid=##,subvol=/@,subvol=@` with `subvol=@`

### Change root

    arch-chroot /mnt /bin/bash

### Hostname

    echo computer_name > /etc/hostname

### Timezone

    ln -s /usr/share/zoneinfo/America/Los_Angeles /etc/localtime

### Locales

    echo LANG=en_US.UTF-8 > /etc/locale.conf

### mkinitcpio Ram Disk

	mkinitcpio -p linux

### Password

    passwd

### Boot Loader

    pacman -S grub
    grub-install /dev/sda
	grub-mkconfig -o /boot/grub/grub.cfg

### Reboot

    exit
	umount -R /mnt
	reboot

At this point, remove the USB flash drive from the server

### Basic Requirements

Start `dhcpcd` on boot

    systemctl enable dhcpcd
	systemctl start dhcpcd

Get `btrfs-progs` to manage snapshots

    pacman -S btrfs-progs

Create snapshots of all subvolumes

	mkdir /btrfs
	mount -t btrfs /dev/sda /btrfs

    btrfs subvolume snapshot -r /btrfs/@ /btrfs/@--$(date --utc --iso-8601="seconds")
	btrfs subvolume snapshot -r /btrfs/@boot /btrfs/@boot--$(date --utc --iso-8601="seconds")
	btrfs subvolume snapshot -r /btrfs/@home /btrfs/@home--$(date --utc --iso-8601="seconds")

	umount -R /btrfs
	rmdir /btrfs

Start 'sshd' on boot

    pacman -S openssh
	systemctl enable sshd
	systemctl start sshd

Install ZSH

    pacman -S zsh zsh-completions
