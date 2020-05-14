

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
# sudo pacman -S xorg xorg-xinit
```

## install mate

Sudo pacman -S mate mate-extra

Yay -S mate-tweak


Install display manager and greeter

Pacman -S lighten lightdm-gtk-greeter

systemctl enable lightdm.service
