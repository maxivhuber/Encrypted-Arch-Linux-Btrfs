# Encrypted-Arch

## Partitioning
##### 3 Filesystems
1. EFI System
2. Linux swap
3. Linux filesystem

## Setup Partitions
mkfs.vfat -F 32 -n EFI /dev/sdx

mkswap -L SWAP /dev/sdx

cryptsetup luksFormat --type luks1 -h sha512 --iter-time 5000 /dev/sdx

cryptsetup luksOpen /dev/sdx cryptroot

mkfs.btrfs -L ROOT /dev/mapper/cryptroot

mount /dev/mapper/cryptroot /mnt

btrfs subvolume create /mnt/@

btrfs subvolume create /mnt/@home

btrfs subvolume create /mnt/@cache

btrfs subvolume create /mnt/@log

btrfs subvolume create /mnt/@spool

btrfs subvolume create /mnt/@temp

btrfs subvolume create /mnt/@snapshots

ls /mnt

umount /mnt

mount -o compress=zstd,subvol=@ /dev/mapper/cryptroot /mnt

mkdir -p /mnt/{home,var/cache,var/log,/var/spool,var/temp,boot/efi}

mount -o compress=zstd,subvol=@home /dev/mapper/cryptroot /mnt/home/

mount -o compress=zstd,subvol=@cache /dev/mapper/cryptroot /mnt/var/cache

mount -o compress=zstd,subvol=@log /dev/mapper/cryptroot /mnt/var/log

mount -o compress=zstd,subvol=@spool /dev/mapper/cryptroot /mnt/var/spool

mount -o compress=zstd,subvol=@temp /dev/mapper/cryptroot /mnt/var/temp

mount /dev/sda1 /mnt/boot/efi

swapon /dev/sdx

df -Th 

## Basic OS installation
pacstrap /mnt base base-devel linux linux-headers linux-firmware intel-ucode
bash-completion btrfs-progs dosfstools grub efibootmgr dhcpcd nano cryptsetup

## Setup OS
genfstab -U /mnt >> /mnt/etc/fstab

arch-chroot /mnt

echo hostname > /etc/hostname

echo LANG=de_DE.UTF-8 > /etc/locale.conf

echo LANGUAGE=de_DE >> /etc/locale.conf

echo KEYMAP=de-latin1-nodeadkeys > /etc/vconsole.conf

ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime

useradd -m -g users -G wheel,audio,video -s /bin/bash username

passwd „name“

EDITOR=nano visudo (uncomment %wheel ALL=(ALL) ALL)

nano /etc/locale.gen (uncomment de_DE.UTF-8)

locale-gen

## Setup GUI
pacman -S xorg-server xorg-xinit xorg-server-xephyr ttf-dejavu gnome
gdm nvidia networkmanager network-manager-applet nm-connection-
editor alsa alsa-tools alsa-utils pulseaudio-alsa pavucontrol cronie ntp
avahi acpid

systemctl enable dhcpcd acpid avahi-daemon cronie ntpd NetworkManager gdm

ntpd -gq

## Setup GRUB and autologin
nano /etc/default/grub (https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Configuring_GRUB_2)

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=arch-crypt

grub-mkconfig -o /boot/grub/grub.cfg

dd bs=512 count=4 if=/dev/random of=/crypto_keyfile.bin iflag=fullblock

chmod 600 /crypto_keyfile.bin

chmod 600 /boot/initramfs-linux*

cryptsetup luksAddKey /dev/sdx /crypto_keyfile.bin

nano /etc/mkinitcpio.conf (https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#Configuring_mkinitcpio)

mkinitcpio -p linux

exit

umount -R /mnt

cryptsetup luksClose cryptroot

reboot

# Setup Snapper
pacman -S snapper
snapper -c root create-config /
btrfs subvolume delete /.snapshots/
mkdir /.snapshots
nano /etc/fstab (automatic mount @snapshots to /.snapshots)
mount -a
chmod 750 /.snapshots
pacman -S grub-btrfs
pacman -S snap-pac
Install package „snap-pac-grub“ from AUR

Snapper can be configured in /etc/snapper/configs/root




















