# Arch Install Guide

Hey, this is a quick guide to install Arch Linux with your own hands 😊
> Trust me, you can install it without those arch-install-scripts. You can flex it!!

**First step**
-----
- Download the Arch Linux ISO from the website → [link](https://archlinux.org/download/)
- Scroll down and click any link from your region listed on the download page.
- Download the file `archlinux-x86_64.iso`.
- Use an image flashing tool like Rufus and flash it to a USB drive.

> Note: 80% of your job is done. The rest 20% is patience and following this guide.

---

## Installation Guide

**1) Keyboard & Internet**

```bash
> loadkeys us
> iwctl
> ping archlinux.org
```

- `loadkeys us` sets your keyboard layout to US.
- `iwctl` it opens wifi tool, Inside it run these one by one:

  ```sh
  > device list
  > station wlan0 scan
  > station wlan0 get-networks
  > station wlan0 connect "YourWIFEName"
  ```
> Note: replace `wlan0` with your device name if different, and `"YourWiFiName"` with your Wi-Fi name. Enter the password when asked.
- Type `exit` to leave iwctl.
- `ping archlinux.org` checks if the internet works. If you see replies, you're connected. Press `Ctrl + C` to stop.

---

**2) Partitioning**

```sh
> cfdisk /dev/sdX
> lsblk
```

- Replace `sdX` with your actual disk (check with `lsblk` — it's usually `sda` or `nvme0n1`).
- Inside `cfdisk`, you will create **three partitions**:

  | Partition | Size                 | Type             |
  | :-------- | :------------------: | :--------------- |
  | EFI       | 512M                 | EFI System       |
  | Swap      | 4G                   | Linux swap       |
  | Root      | the rest of the disk | Linux filesystem |

  > Note: Swap you can have how much ever size you want. but recommended `4GB` if you have a low end device with less ram.
  
- How to do it in cfdisk:
  - Select `[ New ]` → type size → press Enter.
  - Select the partition → `[ Type ]` → choose the right type for each.
  - When done, select `[ Write ]` → type `yes` → `[ Quit ]`.

- Run `lsblk` after to confirm your partition names. They will look like:
  - `/dev/sdX1` → EFI
  - `/dev/sdX2` → swap
  - `/dev/sdX3` → root

> If your disk is NVMe, names look like `/dev/nvme0n1p1`, `/dev/nvme0n1p2`, `/dev/nvme0n1p3`.

---

**3) Formatting**

```sh
> mkfs.ext4 /dev/sdX3
> mkfs.fat -F 32 /dev/sdX1
> mkswap /dev/sdX2
```

- `mkfs.ext4 /dev/sdX3` → formats the root partition as ext4.
- `mkfs.fat -F 32 /dev/sdX1` → formats the EFI partition as FAT32 (required for EFI).
- `mkswap /dev/sdX2` → prepares the swap partition.

> If it asks to confirm, type `yes`.

---

**4) Mounting**

```sh
> mount /dev/sdX3 /mnt
> mkdir -p /mnt/boot/efi
> mount /dev/sdX1 /mnt/boot/efi
> swapon /dev/sdX2
```

- Mount root (`sdX3`) to `/mnt`.
- Create the EFI mount folder and mount the EFI partition (`sdX1`) inside it.
- Turn on swap (`sdX2`).

> Order matters: root first, then EFI, then swap.

---

**5) Base Install**

```sh
> pacstrap /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr nano networkmanager
> genfstab -U /mnt >> /mnt/etc/fstab
> arch-chroot /mnt
```

- `pacstrap` installs the base system, kernel, firmware, bootloader (GRUB), text editor (nano), and NetworkManager.
- `genfstab` writes your mount info into the new system's fstab (so it mounts correctly at boot).
- `arch-chroot /mnt` switches you into the new system.

> From now on, every command runs inside your new install.

---

**6) System Config**

```sh
> ln -sf /usr/share/zoneinfo/Asia/Kolkata /etc/localtime
> hwclock --systohc
```

> Replace `Asia/Kolkata` with your timezone. List available ones with `ls /usr/share/zoneinfo/`.

```sh
> nano /etc/locale.gen
```

- Find the line `#en_US.UTF-8 UTF-8` and **remove the `#`**.
- Save with `Ctrl + O`, Enter, then `Ctrl + X` to exit.

```sh
> locale-gen
```

```sh
> nano /etc/locale.conf
```
- Add this line:

  ```
  LANG=en_US.UTF-8
  ```
- Save and exit.

```sh
> nano /etc/vconsole.conf
```
- Add this line:

  ```
  KEYMAP=us
  ```
- Save and exit.

```sh
> nano /etc/hostname
```
- Type a name for your machine (e.g. `mywife`), save, exit.

---

**7) Users**

```sh
> passwd
```
- Set the **root** password (type it twice; nothing shows while typing — that's normal).

```sh
useradd -m -g wheel -s /bin/bash <username>
passwd <username>
```
- Creates user `<username>` with a home folder and adds them to the `wheel` group.
- Then set that user's password.
> Note: replace with your name or something fancy you would like to keep in `username`.

```sh
EDITOR=nano visudo
```
- Scroll to the line `# %wheel ALL=(ALL:ALL) ALL`.
- **Remove the `#`** so it reads `%wheel ALL=(ALL:ALL) ALL`.
- Save and exit. This lets your user use `sudo`.

---

**8) Bootloader**

```sh
> systemctl enable NetworkManager
> grub-install /dev/sdX
> grub-mkconfig -o /boot/grub/grub.cfg
```

- Enable NetworkManager so internet works after boot.
- `grub-install /dev/sdX` installs GRUB to the **disk** (not a partition — no number at the end).
- `grub-mkconfig` builds the GRUB config file.

---

**9) Wrapping Up**

```sh
> exit
> umount -a
> reboot
```

- `exit` leaves the chroot.
- `umount -a` unmounts everything.
- `reboot` restarts. **Pull out the USB before it boots back up.**

---

**10) Desktop (KDE Plasma)**

After reboot, log in as `root`, then:

```sh
> pacman -S plasma-meta plasma-login-manager
> pacman -S konsole kate firefox
> systemctl enable --now plasmalogin
```

- First line installs Plasma and the login manager.
- Second line installs a terminal, editor, and browser.
- Third line starts the login screen now (and on every boot).

Reboot once more, and you'll see the KDE login screen. Log in as your user and you're done 🎉

---
> Note: After everything is done, all you can do is bless me..... 😊

## Requirements
- An internet connection
- An Arch Linux installation medium (USB)
- A Laptop that works

## Acknowledgement

This guide follows the standard installation flow documented by the [**Arch Linux**](https://archlinux.org/) project, and installs [**KDE Plasma**](https://kde.org/plasma-desktop/) as the desktop environment. All packages referenced come from the official Arch repositories.

## References

- [Arch Wiki – Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
- [Arch Wiki – General Recommendations](https://wiki.archlinux.org/title/General_recommendations)
- [Arch Wiki – KDE Plasma](https://wiki.archlinux.org/title/KDE)

## License

```
MIT License

Copyright (c) 2026 Surendra S

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
