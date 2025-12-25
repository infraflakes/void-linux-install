# How I manually setup Void Linux (uses XFCE image)

### Partitioning and install base system

Mount main dir as ext4 and /mnt

Mount boot dir as vfat and /mnt/boot/efi

Make sure network connected then to install base system:

```
sudo xbps-install -Sy -R https://repo-fastly.voidlinux.org/current -r /mnt base-system
```

After that generate fstab:

```
sudo su
sudo xgenfstab -U /mnt > /mnt/etc/fstab
```

```
sudo xchroot /mnt /bin/bash
```

### Setting up the repo

After chroot, set fastly for the main repo:

```
cp /usr/share/xbps.d/00-repository-main.conf /etc/xbps.d/
vi /etc/xbps.d/00-repository-main.conf

repository=https://repo-fastly.voidlinux.org/current
```

Then install other repo and xtools:

```
sudo xbps-install -Su xtools void-repo-multilib void-repo-nonfree
```

Set fastly for other repo:

```
cp /usr/share/xbps.d/10-repository-multilib.conf /etc/xbps.d/
cp /usr/share/xbps.d/10-repository-nonfree.conf /etc/xbps.d/

vi /etc/xbps.d/10-repository-multilib.conf
repository=https://repo-fastly.voidlinux.org/current/multilib
vi /etc/xbps.d/10-repository-nonfree.conf
repository=https://repo-fastly.voidlinux.org/current/nonfree
```

### Setting up other essentials:

Set hostname:

```
vim /etc/hostname
serein
```

Set locale:

```
vim /etc/default/libc-locales
```

uncomment the en_US locale then save

```
xbps-reconfigure -f glibc-locales
```

Set time (for examples Asia Ho_Chi_Minh):

```
sudo ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime
```

Create passwd

```
passwd
```

Create user

```
useradd -mG wheel nixuris
passwd nixuris
```

install some basic pkgs:

```
xi pipewire elogind base-devel curl wget brightnessctl vim grub efibootmgr dbus polkit rtkit fish-shell xorg xinit NetworkManager fastfetch
```

Edit sudoer:

```
EDITOR=vim visudo
```

Install grub:

```
xbps-install -S grub-x86_64-efi
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id="Void"
grub-mkconfig -o /boot/grub/grub.cfg
```

Enable some services:
```
ln -s /etc/sv/polkitd /var/service/
ln -s /etc/sv/rtkit /var/service/
ln -s /etc/sv/dbus /var/service/
ln -s /etc/sv/NetworkManager /var/service/
```

Set 1.1.1.1 DNS:

```
vim /etc/resolv.conf
nameserver 1.1.1.1
```

Set up pipewire:

```
ln -s /etc/sv/elogind /var/service
mkdir -p /etc/pipewire/pipewire.conf.d
ln -s /usr/share/examples/wireplumber/10-wireplumber.conf /etc/pipewire/pipewire.conf.d/
ln -s /usr/share/examples/pipewire/20-pipewire-pulse.conf /etc/pipewire/pipewire.conf.d/
```

Reconfigure for stability:

```
xbps-reconfigure -fa
```

Exit chroot then umount

```
sudo umount -R /mnt
```

### Post install

Connect with nmtui (make sure dbus and networkmanager is running)

Install nvidia:

```
xi nvidia
echo 'options nvidia NVreg_EnableGpuFirmware=0' | sudo tee /etc/modprobe.d/nvidia-gsp.conf
sudo dracut --force && sudo reboot
```

If brightnessctl need super user:

```
sudo chmod +s /usr/bin/brightnessctl
```

Install nix:

```
xi nix
sudo ln -s /etc/sv/nix-daemon /var/service/
sudo sv up nix-daemon
```

Set up powerprofiles:

```
xi power-profiles-daemon
sudo ln -s /etc/sv/power-profiles-daemon /var/service
sudo sv up power-profiles-daemon
```

Set up flatpak:

```
xi flatpak
flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak --user install flathub app.zen_browser.zen
flatpak override --user --filesystem=~/Downloads app.zen_browser.zen
```

Steam:

```
xi MangoHud steam libgcc-32bit libstdc++-32bit libdrm-32bit libglvnd-32bit mesa-dri-32bit
```

Add this to .steam/steam/steam_dev.cfg

```
unShaderBackgroundProcessingThreads 20
```
