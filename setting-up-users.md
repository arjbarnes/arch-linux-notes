# Arch Linux Installation (User Account Setup)

## Create a new user account
Create the user account using:
```
# useradd -m -s /bin/bash <username>
```
The -m flag creates a home directory for the user at /home/<username>.
The -s flag specifes the default shell for the user (this must be one that is listed in /etc/shells).
Can also specify groups that the user should belong to using the -G flag (e.g. -G <group1>, <group2>) 
Set the password for the new user:
```
# passwd <username>
```

## Install and configure sudo
Install the sudo package using:
```
pacman -S sudo
```
Edit the sudo configuration using:
```
EDITOR=vim visudo
```
N.B. it is necessary to use the 'visudo' command to edit the configuration file for sudo (Which is /etc/sudoers).  Need to pass the EDITOR variable as visude uses 'vi' by default and we only have 'vim' installed.

There are lots of ways to configure sudo - for me, I'll configure sudo so that any members of the 'wheel' group can run commands as root with sudo by entering their own password.  To do this, uncomment the following line:
```
> %wheel ALL=(ALL) ALL
```
I'm also going to change the password prompt timeout so that it will stay visible until a password is entered by adding:
```
> Defaults passwd_timeout=0
```
For fun, I'm also going to enable the insults feature which provides a humerous insult when a password is incorrectly entered by adding:
```
> Defaults insults
```
Next, add any users that should be able to run commands as root through sudo to the 'wheel' group:
```
# usermod -aG wheel <username>
```
Can check which groups the user is a member of using:
```
# groups <username>
```

## Disable the root account

It is best to logout and run the commands to disable the root account from your user account in order to ensure that your 
user account has sudo accesss.  So first logout of the root account and login to your user account:
```
# exit
```
Lock the root account and remove the root password:
```
# sudo passwd -ld root
```
The '-l' flag locks the root account (but leaves the password set).
The '-d' flag removes the root password.
