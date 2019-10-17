

## Install a display driver
Check what video card is present using:
```
lspci
```

For virtualbox:
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
# sudo pacman -S sddm plasma-desktop
```
```
# echo "exec startkde" > ~/.xinitrc
```
```
sudo pacman -S konsole dolphin firefox kate
```
