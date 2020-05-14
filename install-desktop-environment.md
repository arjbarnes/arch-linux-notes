

## Install a display driver
Check what video card is present using:
```
lspci | grep -e VGA -e 3D
```
For NVidia
```
sudo pacman -S nvidia nvidia-libgl mesa

```


## Install the X display server
```
# sudo pacman -S xorg xorg-server xorg-xinit 
```

## install mate

Sudo pacman -S mate mate-extra

Yay -S mate-tweak


Install display manager and greeter

Pacman -S lightdm lightdm-gtk-greeter

systemctl enable lightdm.service


## Install Plymouth

yay -S plymouth
sudo pacman -S ttf-dejavu
sudo vim /etc/mkinitcpio.conf
> add 'plymouth' into the HOOKS section after 'base' and 'udev'
> replace 'encrypt' in the HOOKS section with 'plymouth-encrypt'
> add 'i915' into the MODULES section
sudo vim /boot/loader/entries/arch.conf
> add 'splash' at the end of the 'options' entry
mkinitcpio -p linux

sudo systemctl disable lightdm.service
sudo systemctl enable lightdm-plymouth.service

## Networking
networkmanager
network-manager-applet
nm-applet
systemctl enable NetworkManager

## Firewall
sudo pacman -S ufw
sudo systemctl enable ufw
sudo systemctl start ufw
sudo ufw enable
sudo ufw status verbose

## Audio
pacman -S pulseaudio pulseaudio-alsa

## Syncthing
sudo pacman -S syncnthing syncthing-gtk
yay -S syncthingtray
systemctl --user enable syncthing.service
systemctl --user start syncthing.service
syncthingtray

## Install Firefox
pacman -S firefox
pacman -S vlc audacious
