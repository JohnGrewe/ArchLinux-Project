I installed arch linux onto VMware
I added firmware="efi" to Other Linux 5.x kernel 64-bit.vmx
boot failed to launch after this, I chose discard on the vmware machine and it booted up like
example

## 1.6 Verify the boot mode
I used the command
```
cat /sys/firmware/efi/fw_platform_size and got 64
```

This means it booted in UEFI mode and have a 64 bit UEFI

## 1.7 Connect to the internet
I used the command ip link and it showed I have and internet connection
I pinged archlinux.org and got responses back

## 1.8 Update the System Clock
I used the command: 
```timedatectl```
It gave back the date of thursday when it is wednesday but it did work

## 1.9 Partition the disks
I used the command 
fdisk -l
I used the command 
fdisk /dev/sda
I used m for help, then I did p (primary), n(add a new partition), 1, and +500M for the boot drive
Partitioned the other part for 19,5 GB
Saved and exited with w

## 1.10 Format the partitions
I used on the EFI sda1 partition (500M)
mkfs.fat -F 32 /dev/sda1
no warnings
I used mkfs.ext4 /dev/sda2
go a /dev/sda2 contains 'DOS/MBR boot sector, proceed anyway?'
type y
No problems so far

## 1.11 Mount the file systems
mount --mkdir /dev/sda1 /mnt/boot
no surprises
mount /dev/sda2 /mnt
no surprises

## 2.1 Select the Mirrors
Dont have to do in this specific case

## 2.2 Install Essential Packages

## 3.1 Fstab
genfstab -U /mnt >> /mnt/etc/fstab

## 3.2 Chroot
arch-chroot /mnt

## 3.3 Time
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc (assumes hardware clokc set to UTC?)

## 3.4 Localization
nano: command not found
pacstrap: command not found
installed nano, pacman worked and pacman -Sy nano
nano /etc/local.gen
uncommented en_US.UTF-8 UTF-8
locale-gen
nano /etc/locale.conf
added LANG=en_US.UTF-8
nano /etc/vconsole.conf
KEYMAP=us
pacstrap -K /mnt base linux linux-firmware

## 3.5 Network Configuration
nano /etc/hosts
127.0.0.1 localhost
::1 localhost
127.0.1.1 archjack.localdomain archjack
pacman -S networkmanager
systemctl enable NetworkManager
systemctl start NetworkManager
running in chroot, ignoring command 'start'

## 3.6 Initramfs
Not Modifying since not using LVM or RAID

## 3.7 Root Password
passwd
password updated successfully

## 3.8 Boot Loader
pacman -S grub efibootmgr
Y
grub-install --target=x86_64-efi --efi-directory=/boot
error: /boot doesnt look like an EFI partition
mount /dev/sda1 /mnt/boot
/mnt/boot mount point does not exist
mkdir -p /mnt/boot
mount /dev/sda1 /mnt/boot
grub-install --target=x86_64-efi --efi-directory=/mnt/boot --bootloader-id=GRUB
installation finished. no error reported
grub-mkconfig -o /boot/grub/grub.cfg
/usr/bin/grub-mkconfig: line 270 /mnt/boot/grub/grub.cfg.new: no such file or directory
mount | grep /mnt/boot
/dev/sda1 on /mnt/boot
/dev/sda1 on /mnt/boot
nano /etc/hostname
archjack
mkdir -p /mnt/boot/grub
grub-mkconfig -o /boot/grub/grub.cfg
done

## 4 Reboot
exit
done
unmount -R /mnt
weird message from zsh:
zsh: correct 'unmount' to 'unmountzsh: correct 'unmount' to 'umount' [nyae]?
nothing
command not found: unmount
ITS umount -R /mnt OMG IM SO DUMB
/mnt/mnt/boot: not mounted
reboot


## GRUB Troubles
GNU GRUB version 2:2.12rc1-5
grub>
ls
(hd0) (hd0,msdos2) (hd0, msdos1) (cd0) (cd0, msdos2)
set root=(hd0,msdos1)
linux /vmlinux-linux root=/dev/sda1
error: /vmlinux-linux cant be found
grub> ls (hd0,msdos1)/
boot/ efi/ grub/
grub> ls (hd0,msdos2)/
lost+found/ var/ dev/ run/ etc/ tmp/ sys/ proc/ usr/ bin boot/ home/ lib lib64 mnt/ opt/ root/ sbin
srv/
set root=(hd0, msdos2)
done
grub> linux /boot/vmlinuz-linux root=/dev/sda2
grub> initrd /boot/initramfs-linux.img
boot
LOGGED IN OH MY GOD YESSSSS!!!!


## GRUB Troubles Part 2 Electric Boogaloo
Opened up to GNU GRUB again... AHHHHHHHHHHHH
ls
(hd0) (hd0, msdos2) (hd0, msdos1)
grub> set root=(hd0, msdos2)
grub> linux /boot/vmlinuz-linux root=/dev/sda2
error: disk '(hd0,' not found.
ls again
ls (hd0,msdos1)/
boot/ efi/ grub/
grub> set root=(hd0,msdos2)
no error
root archjack login
grub-install: error: failed to get canonical path of '/boot/efi'
lsblk
sda1 not mounted to /boot/efi
mount | grep /boot/efi
lsblk, shows still not mounted
mkdir -p /boot/efi
mount /dev/sda1 /boot/efi
lsblk confirms mount
blkid /dev/sda1
UUID = F129-8ABB
nano /etc/fstab
UUID=F129-8ABB /boot/efi vfat umask=0077 0 2
grub> linux /boot/vmlinuz-linux root=/dev/sda2
grub> initrd /boot/initramfs-linux.img
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
tested with mount -a
nothing happened good to go

## Okay, now that I got that out of the way, time to
install a desktop Environment
I decided to choose GNOME as it seemed like the most versatile and user friendly of the
desktop environments. I thought about dwm, but ultimately decided on GNOME
first, pacman -Syu
pacman -S gnome
I did all defaults for download
pacman -Qi networkmanager
pacman -S networkmanager
pacman -S gnome-shell-extensions
pacman -S gnome-control-center
pacman -S sudo
sudo pacman -S git
sudo pacman -S gnome-extra
sudo systemctl enable gdm.service
sudo systemctl start gdm.service
Started in GNOME
username: root
password: ha you thought
I GOT A DESKTOP ENVIRONMENT!!!
sudo useradd -m adminjack
sudo passwd adminjack
set password
usermod -aG wheel adminjack
	nanoed into wheel and made wheel group have admin privileges
sudo pacman -S man-db man-pages
reboot

## GRUB PART 3
I forgot to configure GRUB.....
set root=(hd0,msdosX)
linux /boot/vmlinuz-linux root=/dev/sda2
initrd /boot/initramfs-linux.img
boot
Got back in to GNOME, now to fix this once and for all
Fix booting into GRUB
grub-mkconfig -o /boot/grub/grub.cfg
dobe
grub-install /dev/sda
installation finished, no error reported
fsck
/dev/sda2 is mounted.
reboot
BOOTED INTO GNOME YESS ITS OVER I DID IT

## Now For Some Customization


```
pacman -S networkmanager
```

```
pacman -S gnome-shell-extensions
```

```
pacman -S curl
```

```
sudo nano /etc/pacman.conf
```
	```
	[multilib]
	Include = /etc/pacman.d/mirrorlist
	```

```
sudo pacman -S steam
```
	Set up steam on ArchLinux

```
sudo pacman -S mpv
```

```
sudo pacman -S git
```

```
sudo pacman -S git base-devel
```

```
git clone https://aur.archlinux.org/yay.git
```

```
cd yay
```

```
makepkg -si
```

```
yay -S xwinwrap-bzr
```

```
sudo EDITOR=nano visudo
```
	#%wheel ALL=(ALL) ALL

```
mkdir /home/adminjack/build
```
```
sudo chown adminjack:adminjack /home/adminjack/build
```

```
sudo pacman -Syu base-devel xorg-server-devel libx11 libxext
```

```
git clone https://github.com/ujjwal96/xwinwrap.git
```
```
cd xwinwrap
```
```
make
```
```
sudo make install
```
```
make clean
```

```
yay -S yt-dlp
```

```
yt-dlp https://www.youtube.com/watch?v=7yzImCTv7A0
```

```
ffmpeg -i Bowling Alley screens when you get a Strike [7yzlmCTv7A0].webm rightsize.webm
```

At this point I was trying to open the webm with mpv, but realized I dont have hardware accelration or video memory dedicated, so I dedicated some memory and enabled hardware acceleration

reboot

Video worked

```
ffmpeg -i rightsize.webm -vf "fps=30,scale=320:-1:flags=lanczos" -c:v gif -f gif rightsizegif.gif
```

```
sudo pacman -S gifsicle
```

```
xwinwrap -g 1280x800 -ni -s -nf == gifview -w WID rightsizegif.gif -a
```

Works












## Fulfilling the Rubric



```
sudo pacman -S zsh
```

```
sudo useradd -m codi
```

```
sudo passwd codi
```

```
sudo chage -d 0 codi
```

```
ssh adminjack@129.244.245.111
```
	ssh: connect to host 129.244.245.111 port 22: Connection refused

```
sudo pacman -S openvpn
```

```
sudo openvpn --config /etc/openvpn/client/myvpn.ovpn
```
	error

```
changed the .bashrc PS1 to : PS1='\[\033[0;32m\]\u@\h:\w\$'
```

```
Added alias to bashrc, alias c='clear
alias update='sudo apt update && sudo apt upgrade'
```
