# Introduction

Step-by-step guide for
- installing Arch Linux on VirtualBox
- setting up a display server, a window manager, compositor, etc.
- installing necessary apps like mpv, vs code, etc.

This guide assumes you have downloaded an Arch Linux ISO, installed Oracle VirtualBox, and have an internet connection.

&nbsp;

## VirtualBox VM Settings

1. Click on 'New'. `Type` = 'Linux' and `Version` = 'Arch Linux (64-bit)'.
2. `Memory` = 2048 MB. (I have 8 GB total.)
3. Create a VDI virtual hard disk, dynamically allocated, 50 GB.
4. A new VM will be created.
5. Right click on it, go to Settings.
6. System:
    * Motherboard: `Enable EFI` and `Harware Clock in UTC Time`.
    * Processor: 2 CPUs
7. Display
    * Video Memory: 128 MB
    * Graphics Controller: VBoxVGA
        * VMSVGA is recommended, but can mess up the guest's screen reso.
    * 3D acceleration enabled.
8. Storage: Add the Arch Linux ISO in Storage Devices/`Controller:IDE`. This will be removed after the install.
9. Make sure the host has an internet connection. Whatever the connection type (Ethernet, WiFi, etc.) for the host will appear as Ethernet for the guest.

&nbsp;

## Installing Arch

Follow the steps in the [Arch Wiki](https://wiki.archlinux.org/title/installation_guide#Set_the_console_keyboard_layout). 

1. Start the VM. Choose installation media (the Arch ISO).

> If there is a kernel panic and the host is Windows, uninstall WSL on host and uncheck any virtualization or hypervisor related feature in `Turn Windows Features On or Off`.

2. Check boot mode (should be EFI in this case), internet connection, and update system clock.
3. Partition the disks. We need an EFI partition, a swap, and a root partition.
    * `fdisk -l` to get the name of the disk to be partitioned (this will be 50 GB in this case).
    * `fdisk path/to/disk`
    * Enter `g` to create GPT partition table.
    * Enter `n` to create a new partition. Last sector: `+550M`, as EFI partition needs to be at least 512 MB.
    * `n`, `+2G` for swap partition.
    * `n`, keep defaults. This will be `/`.
    * `t` to change partition type. Select partition `1`, then select `1` to convert to `EFI System`.
    * `t`, partition `2`, select `19` to convert to `swap`.
    * Enter `p` to check if all partitions are as desired.
    * `w` to write and quit. `q` to quit without writing; use this if you mess up.
4. Format the partitions.
5. Mount root to `/mnt`. Turn on swap.
6. Run `pacstrap /mnt base linux linux-firmware`. Installs the base system, the Linux kernel, and some firmware.

&nbsp;

## Post-Installation

1. Generate `fstab`.
2. `arch-chroot` into `/mnt`.
3. Set time zone (assumes hardware clock is in UTC).
4. Edit `/etc/locale.gen` and create `/etc/locale.conf` file.
> Install `nano` using `pacman -S nano`.
5. Edit `/etc/hostname`. I set my hostname as 'arch-vbox'.
6. Edit `/etc/hosts`. Read [doc](https://wiki.archlinux.org/title/Network_configuration#Local_hostname_resolution).
7. Set root password using `passwd`.
8. Add a user using `useradd -m <username>`, set password. Some packages can't be installed as root, but only as a user.
9. Give user permissions: `usermod -aG wheel,audio,video,optical,storage <username>`.
10. Install sudo: `pacman -S sudo`.
11. Run `EDITOR=nano visudo` and uncomment `wheel ALL = (ALL:ALL) ALL` in the section 'User privilege specification'.
12. Install grub: `pacman -S grub`.
13. `pacman -S bootmgr dosfstools os-prober mtools`
14. Mount EFI partition to `/boot/efi`.
15. `grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck`
16. `grub-mkconfig -o /boot/grub/grub.cfg`.
> `os-prober` might be disabled by `grub` at this stage. Enable to discover OSes on other bootable partitions.
17. Use `pacman` to install `networkmanager` and `git`.
18. `systemctl enable NetworkManager`.
19. Exit and unmount.
```
exit
umount -R /mnt (If this doesn't work use umount -l /mnt)
shutdown now
```
20. In VBox, go to the VM, Settings, then Storage, and remove the Arch ISO (remove attachment).

See [General recommendations](https://wiki.archlinux.org/title/General_recommendations#System_administration) and [List of applications](https://wiki.archlinux.org/title/List_of_applications) in the wiki.

&nbsp;

## GUI

1. Log in as the newly created user.
2. `sudo pacman -S xf86-video-fbdev xorg xorg-xinit nitrogen picom alacritty nemo gedit firefox`
    * Replace `xf86-video-fbdev` with an NVIDIA or AMD display driver if installing on physical.
    * `xorg` is the X display server.
    * `nitrogen` draws wallpapers.
    * `picom` is a compositor, a fork of `compton`.
    * `alacritty` is a terminal emulator.
    * `nemo` is a file manager, forked from `nautilus`.
> Install these programs individually if they won't install together.
3. Get [paru](https://github.com/morganamilo/paru), an AUR helper. `base-devel` is needed for `makepkg`.
```
sudo pacman -S --needed base-devel
git clone https://aur.archlinux.org/paru.git
cd paru
makepkg -si
```
4. Install `awesome` WM: `paru awesome`.
5. `cp /etc/X11/xinit/xinitrc /home/username/.xinitrc`
6. Delete the last 5 lines of `.xinitrc` where `twm` and others launch, and type `exec awesome`.
7. Install `rofi`: `pacman -S rofi`

&nbsp;

## AwesomeWM 
1. Edit `rc.lua`
    * Change order of layouts
    * Add
```
awful.spawn.with_shell("nitrogen --restore")
awful.spawn.with_shell("picom")
```
2. Set a wallpaper with `nitrogen`. This will be restored when `awesome` starts.
1. Have `awesome` run `rofi` as the default prompt: look for code under the comment `--Prompt` in `rc.lua`. Change the function to
```
awful.spawn("rofi -show run")
```
4. Add shortcuts to applications (like Firefox) using `awful.spawn()` as above. Super (Win) + S to see keybindings.
5. Remove titlebars by commenting out the code block under the comment `-- Add titlebars to normal clients and dialogs`.

&nbsp;












