Arch Install Guide
===================

A beginner-friendly guide to installing Arch Linux on your Bare metal and i assume that you have already booted into the installer.iso

Usage
-----

**Keyboard & internet**
```sh
loadkeys us
iwctl
ping archlinux.org
```
Set your layout, connect to Wi-Fi from inside `iwctl`, then confirm the connection is up. `Ctrl + C` to stop the ping.

**Partitioning**
```sh
cfdisk
lsblk
```
Create your partitions (EFI, root, swap), then double-check them once.

**Formatting**
```sh
mkfs.ext4 /dev/sdX3
mkfs.fat -F 32 /dev/sdX1
mkswap /dev/sdX2
```
Root as ext4, EFI as FAT32, swap prepared.

**Mounting**
```sh
mount /dev/sda3 /mnt
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
swapon /dev/sda2
```
Mount root to `/mnt`, Mount EFI into `/mnt/boot/efi`, swap turned on.

**Base install**
```sh
pacstrap /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr nano networkmanager
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```
Install linux kernel with base, firmware, bootloader which is `grub` and `networkmanager`.

**System config**
```sh
ln -sf /usr/share/zoneinfo/Asia/Dubai /etc/localtime
hwclock --systohc
```
Set the local-time to your timezone and sync the clock time to your hardware.
