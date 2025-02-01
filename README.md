# Betims arch linux install guide w/ hyprland!

* This is going to be more of a walkthrough / guide on how i did it, you can make tweaks to things I've done, but since i've configured for an alienware laptop its a bit more specific than id like it to be.
* so ill mention where i had to make weird tweaks so that you can make your own choices when we come to that road


## Prereqs
* bootable arch linux usb: https://archlinux.org/download/
* UEFI system 
* internet connection (Ethernet is easiest, Wifi is available tho just more config)

### ARCH USB INSTALL

1. Connect to wifi (if needed)
```
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect [SSID]
exit
```
* if you don't know your wifi devices name, run ```ifconfig```

2. Partitioning

to make this a bit easier i used cfdisk since it had a gui like menu

* if you're unsure what your device ssd is called run ```lsblk```


Our paritioning scheme is gonna be simple but you can change it up if you want for your prefrences

* EFI Partition 512M
* Swap 2-4GB (can make this equal size to ur current ram for hibernation)
* Root remaining space

```
cfdisk /dev/sda (replace sda w/ your disk)
```

this is an example layout of your ssys
```
device      size    type
/dev/sda1   512MB   EFI System
/dev/sda2   8GB     Linux Swap
/dev/sda3   490GB   Linux filesystem
```

#### Formatting & mounting

```
mkfs.fat -F32 /dev/sda1          # EFI
mkswap /dev/sda2                 # Swap
mkfs.ext4 /dev/sda3              # Root (or btrfs if preferred)

# Mount partitions
mount /dev/sda3 /mnt
mount --mkdir /dev/sda1 /mnt/boot
swapon /dev/sda2
```

#### Installing the base system 
* basic kernel stuff to get our OS running

```
pacstrap -K /mnt base linux linux-firmware nano sudo grub networkmanager
```
* note i went for a hardened linux kernel install to look at what different kernels are available please check this out:https://wiki.archlinux.org/title/Kernel 
* you would have to replace linux: kernel-choice

#### Generate fstab

```
genfstab -U /mnt >> /mnt/etc/fstab 
```

### Chrooting into your system

Essentially you're entering your FS, and its gonna be like you're logged in as root, but since nothings setup yet, we'll need to do so.

```
arch-chroot /mnt

# Timezone
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc

# Locale
nano /etc/locale.gen  # Uncomment your locale (e.g., en_US.UTF-8)
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Hostname
echo "arch-yourhostnamehere" > /etc/hostname

# Root password
passwd
```

#### Bootloader (GRUB)
Theres some prereqs with this that you might need, let me introduce you to the package manager in arch, pacman!

```
pacman -S efibootmgr os-prober
```

Now we can install grub to the EFI partition
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```
next we need to regenerate the grub config so that when we boot we can boot into arch
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

#### Rebooting your system

* if everything has gone right, which is very unusual to happen the 1st time around (always ask for help!) then you'll be able reboot into your system

```
exit
umount -R /mnt
reboot
```

### Setting up the desktop for daily use

Currently right now you are in arch w/out a usb iso, which is great! The next part is to install a desktop manager, I've use gnome, plasma and they're great but i went along with hyprland becasue it looks nice, especially this prebuilt install from this repo.

#### Setting your user in the sudoers file

right now your user cant use sudo, its annoying but luckily you can log into root
```
user: root
pass: yourrootpass
```

* now lets add you to the wheel group
```
sudo usermod -aG wheel user

orr

sudo gpasswd -a user wheel
```

* that may or may not work for you, i had to uncomment some things from the sudoers file

```
visudo
```

i uncommented these line

```
root ALL=(ALL:ALL) ALL
%wheel ALL=(ALL:ALL) ALL
%sudo ALL=(ALL:ALL) ALL
```

only after that did it start working for me (technically insecure but more convince)

#### Installing yay (AUR Helper)

Install dependencies:
```
sudo pacman -S base-devel git
```

Clone the yay repository:

```
git clone https://aur.archlinux.org/yay.git
cd yay
```

Build and install yay:
```
makepkg -si
```

Now you can install packages using yay

```
yay -S package-name
```

from here on out, pacman is usually gonna be CLI packages and Yay will be used more for full blown applications

#### Installing a web browser

You could use firefox, but i perfer brave, and its so easy to install with Yay

```
yay -S brave-bin
```

#### Installing fan control (Alienware)

My alienware laptop has weird fan controls, so i found i8ktuils works the best for me on dell/alienware laptops, you might have different dependencies needed or not needed.

* install i8kutils
```
yay -S i8kutils
```

The next thing i had to do to automate it was make a conf file for i8ktuils

```
sudo nvim /etc/i8kutil.conf
```
* note any text editor i have used here i've installed in pacman

inside the file add these:
```
set config(autofan) 1
set config(autoac) 1
```
start the service w/ systemctl
```
sudo systemctl start i8kmon
sudo systemctl enable i8kmon
```

#### Installing a network manager (temporarily until hyprland desktop manager)

```
sudo pacman -S networkmanager
```

```
sudo systemctl enable NetworkManager
sudo systemctl start NetworkManager
```

* run the network manager and connect w/ wifi!
```
nmtui
```

### Installing hyprland

I personalyl used JaKooLit's arch-hyprland install and had some hiccups with it, but figured out how it worked for me.

If it installs all the dot files for you then you can skip some extra steps, but if it doesn't then i'll go over the steps


link to the repo: https://github.com/JaKooLit/Arch-Hyprland?tab=readme-ov-file

```
git clone --depth=1 https://github.com/JaKooLit/Arch-Hyprland.git ~/Arch-Hyprland
cd ~/Arch-Hyprland
chmod +x install.sh
./install.sh
```

after you've gone through the install you should reboot, but it wont look quite exactly how its supposed to.

* you'll log into hyprland and then type (windows) q to open a terminal and type this in

```
git clone --depth=1 https://github.com/JaKooLit/Hyprland-Dots.git
cd Hyprland-Dots

chmod +x copy.sh
./copy.sh
```

after that woot woot you got the basics done, theres probably still a lot more you would wish to have, but thats enough to get you started!

