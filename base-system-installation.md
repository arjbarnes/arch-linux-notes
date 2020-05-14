# Arch Linux Installation (Basic System)

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

efivar -l

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

## Setup hard drive partitions for the installation
Identify the correct hard drive using:
```
# fdisk -l
```
Run the fdisk utility on the hard drive that you want to install to (probably /dev/sda):
```
# fdisk /dev/sda
```
/dev/nvme0n1


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
/dev/nvme0n1p2

Open the encrypted LVM partition using:
```
# cryptsetup open --type luks /dev/sda2 lvm
```
/dev/nvme0n1p2 luks


## Setup the LVM and logical partitions
```
# pvcreate /dev/mapper/lvm
```
/luks


```
# vgcreate system /dev/mapper/lvm
```
<vgname> /dev/mapper/luks

```
# lvcreate -L 30GB system -n root
```
```
# lvcreate -l 100%FREE system -n home
```


## Create filesystems for the installation
```
# mkfs.fat -F32 /dev/sda1
```
/dev/nvme0n1p1

```
# mkfs.ext4 /dev/mapper/system-rooth
```
```
# mkfs.ext4 /dev/mapper/system-home
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
/dev/nvme0n1p1
```
# mkdir /mnt/home
```
```
# mount /dev/mapper/system-home /mnt/home
```

## Install the system 
```
# pacstrap /mnt base base-devel linux linux-lts linux-headers linux-lts-headers linux-firmware intel-ucode lvm2 netctl wpa_supplicant dhcpcd dialog ppp vim
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
Specify the correct locale in /etc/locale.conf:
```
# vim /etc/locale.conf
> LANG=en_GB.UTF-8
```
Uncomment the correct locale in /etc/locale.gen (for me, this is 'en_GB.UTF-8 UTF-8':
```
# vim /etc/locale.gen
```
Generate the localisation information:
```
# locale-gen
```

## Specify the correct timezone for the system
Set the timzone:
```
# ln -sf /usr/share/zoneinfo/Europe/London/ /etc/localtime
```
Generate /etc/adjtime
```
# hwclock --systohc
```

## Specify a hostname for the system
```
# vim /etc/hostname
> <hostname>
```
```
# vim /etc/hosts
> 127.0.0.1 localhost
> ::1 localhost
> 127.0.1.1 <hostname>.<localdomain> <hostname>
```

## Set a root password
```
passwd
```

## Create a swap space
I will create a swap space using a file rather than a hard drive partition as it is more flexible (can be more easily resized later on if needed).  Create the swap file using:
```
# fallocate -l 8G /swap
# chmod 600 /swap
# mkswap /swap
```
Now add the swap file to the /etc/fstab file so that it will be automatically mounted:
```
# echo '# swap file' >> /etc/fstab
# echo '/swap none swap sw 0 0' >> /etc/fstab
```
Check the contents of the fstab file using:
```
# cat /etc/fstab
```

## Install and configure the bootloader
```
# bootctl --path=/boot/ install
```
```
# vim /boot/loader/loader.conf
> clear
> default arch
> timeout 2
> editor no
> auto-entries no
```
Setting editor to no prevents commands being entered in the bootloader screen and therefore improves security.
```
# vim /boot/loader/entries/arch.conf
> title Arch Linux (LTS)
> linux /vmlinuz-linux-lts
> initrd /intel-ucode.img
> initrd /initramfs-linux-lts.img
> options cryptdevice=UUID=<XXXXXXXXXX>:lvm root=/dev/system/root rw quiet
```
N.B. need to insert the UUID of the disk partition with the LVM on it (/dev/sda2) in place of <XXXXXXXXXX> - a shortcut to insert this within vim is to use the following command:
```
: read ! blkid /dev/sda2
```
which will insert hte output from that command onto a line in the file, which can then be edited to remove everything other than the UUID.

## Install and Configure the initial ram disk
Configure additional hooks for 'keymap', 'encrypt' and 'lvm2' and re-order the hooks so that 'keyboard' and 'keymap' appear before 'encrypt':
```
# vim /etc/mkinitcpio.conf
> HOOKS="[...] block keyboard keymap encrypt lvm2 filesystems [...]"
```
Keymap needs to be before encrypt in the list of hooks, otherwise a US keymap will be used for password entry.  Ensure that the correct locale and keymap are specified before generating the initial ram disk, otherwise the wrong keyboard layout will be used (which will make decrypting the lvm harder!).

Generate the initial ram disk:
```
# mkinitcpio -p linux-lts
```

## Reboot into the new system
The basic system should now be installed, so can reboot into it using:
```
# exit
# umount -R /mnt
# reboot
```
