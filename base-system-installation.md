# Arch Linux Installation

## Set an appropriate keyboard layout

To list the available keyboard layouts use:
```
# ls /usr/share/kbd/keymaps/**/*.map.gz
```

Load the appropriate layout (in my case this is 'uk') using:
```
# loadkeys uk
```

## Verify that the boot mode is EFI:
```
# ls /sys/firmware/efi/efivars
```
If this directory does not exist, the system may be booted in BIOS or CSM mode instead.

## Connect to the Internet
If a network cable is plugged in, the system may already be connected.  Otherwise, can connect to  WiFi network using:
```
# wifi-menu
```
Can check internet connectivity using:
```
# ping www.archlinux.org
```

## Set the system clock
```
# timedatectl set-ntp true
```
```
# timedatectl set-timezone Europe/London
```
```
# timedatectl
```

## Setuo hard drive partitions for the installation
Identify the correct hard drive using:
```
# fdisk -l
```
Run the fdisk utility on the hard drive that you want to install to (probably /dev/sda):
```
# fdisk /dev/sda
```

Set the disk to use a GUID partition table (GPT) using the 'g' command.

Create a 512MB partition for the EFI using the 'n' command (N.B. can specify a size, such as +512M, when asked for the last sector).

Change the type of the EFI partition to the 'EFI System' using the 't' command ('EFI System' is option '1').

Create an LVM partition for the system to be installed to using the 'n' command (N.B. can just use the default value when asked for the last sector to use all the remining space).

Change the type of the LVM partition to be 'Linux LVM' using the 't' command (N.B. 'Linux LVM' is option '31').

Write the changes to the disk using the 'w' command.

Can check that the partitions have been setup correctly by using `fdisk -l` again.

## Encrypt the LVM

Encrypt the LVM partition using:
```
# cryptsetup luksFormat /dev/sda2
```
Open the encrypted LVM partition using:
```
# cryptsetup open --type luks /dev/sda2 lvm
```

## Setup the LVM and logical partitions
```
# pvcreate /dev/mapper/lvm
```
```
vgcreate system /dev/mapper/lvm
```
```
lvcreate -L 30GB system -n root
```
```
lvcreate -l 100%FREE system -n home
```
```
modprobe dm_mod (is this necessary?)
```
```
vgscan
```
```
vgchange -ay
```

## Create filesystems for the installation
```
# mkfs.fat -F32 /dev/sda1
```
```
mkfs.ext4 /dev/mapper/system-root
```
```
mkfs.ext4 /dev/mapper/system-home
```

## Mount the filesystems
```
# mount /dev/mapper/system-root /mnt
```
```
# mkdir /mnt/boot
```
```
# mount /dev/sda1 /mnt/boot
```
```
# mkdir /mnt/home
```
```
# mount /dev/mapper/system-home /mnt/home
```

## Install the system 
```
# pacstrap /mnt base base-devel linux-lts linux-lts-headers lvm2 vim
```
I have chosen to use the long-term support (LTS) linux kernel as it may be more stable.  Can use the latest stable linux kernel instead by replacing the 'linux-lts' and 'linux-lts-headers' packages with the 'linux' and 'linux-headers' packages - it will also be necessary to change some of the following commands to refer to 'linux' rther than 'linux-lts'. 

The 'vim' package isn't strictly necessary.  However, it will be necessary to install a text editor in order to edit configuration files.  

## Create an 'fstab' file
The fstab file tells the installed operating system how the hard drive partitions should be mounted.  The followwing command will generate this by looking at how you have mounted the partitions in the earlier steps of the installation and put this informtion into fstab file:
```
# genfstab -p /mnt >> /mnt/etc/fstab
```
Can check what has been created using:
```
# cat /mnt/etc/fstab
```

## Launch into the new system (without restarting)
```
# arch-chroot /mnt
```

## Configure the system to use the correct keyboard mapping
When the system is rebooted, the keyboard mapping that was specified earlier will be lost.  Can persistently specify the keyboard mapping using:
```
# vim /etc/vconsole.conf
> KEYMAP=uk
```
Again, replace 'uk' with the appropriate keyboard mapping as identified earlier on in the installation.

## Set the correct localisation for the system 

## Specify the correct timezone for the system

Set the time zone:

