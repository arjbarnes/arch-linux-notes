

## Install a display driver
Check what video card is present using:
```
lspci
```

For Intel:
```
sudo pacman -S xf86-video-intel-libgl mesa
```

For NVidia:
```
sudo pacman -S nvidia-lts nvidia-libgl mesa
```

For VirtualBox:
```
sudo pacman -S virtualbox-guest-utils virtualbox-guest-modules-arch mesa mesa-libgl
```

## Install the X display server
```
# sudo pacman -S xorg xorg-xinit
```

## Install a Desktop Manager and Environment

### KDE
```
# sudo pacman -S sddm 
```
```
# sudo systemctl enable sddm
```

## Install some core applications
```
sudo pacman -S konsole dolphin firefox kate
```
