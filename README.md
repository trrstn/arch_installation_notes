# Arch Linux Installation with systemd-boot
Arch Linux installation with systemd-boot.
### Bootable flash drive
Download the latest ISO at https://www.archlinux.org/download/ and create a bootable drive following this:
_replace sdX with your usb drive's disk number with: lsblk_ 
```
dd if=/*path_to_arch*/arch_linux.iso of=/dev/sdX status=progress oflag=sync bs=4M 
```
### First Steps
Check if you're system is running in UEFI mode with:
```
ls /sys/firmware/efi
```
If there is any content in this folder you are running in UEFI mode.

Check if there is an internet connection.
```
ping google.com
```
Tether your android phone into your machine and do the following:
```
❯ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: enp2s0f1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN mode DEFAULT group default qlen 1000
    link/ether 98:28:a6:14:36:83 brd ff:ff:ff:ff:ff:ff
3: wlp3s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DORMANT group default qlen 100
```
Find the connection and connect with:
```
dhcpcd enp2s0f1*
```
### Disk and partition
```
gdisk /dev/sda/
```
```
GPT fdisk (gdisk) version 1.0.1

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): Y

Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-242187466, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-242187466, default = 242187466) or {+-}size{KMGTP}: +512M
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): EF00
Changed type of partition to 'EFI System'

Command (? for help): n
Partition number (2-128, default 2): 
First sector (34-242187466, default = 1050624) or {+-}size{KMGTP}: 
Last sector (1050624-242187466, default = 242187466) or {+-}size{KMGTP}: 
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/sda: 242187500 sectors, 115.5 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 9FB9AC2C-8F29-41AE-8D61-21EA9E0B4C2A
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 242187466
Partitions will be aligned on 2048-sector boundaries
Total free space is 2014 sectors (1007.0 KiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048         1050623   512.0 MiB   EF00  EFI System
   2         1050624       242187466   115.0 GiB   8300  Linux filesystem

Command (? for help): w
```
```
mkfs.vfat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```
### Mount the partitions
```
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```
### Generating synced and fast mirrorlist
```
pacman -Syy
```
```
pacman -S reflector
```
```
reflector --latest 10 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```
### Install base system
```
pacstrap /mnt base base-devel linux linux-firmware vim
```
### Generate fstab
```
genfstab -pU /mnt >> /mnt/etc/fstab
```
### Chroot into the new system
```
arch-chroot /mnt
```
### Prepare system
```
timedatectl set-timezone Asia/Manila
```
uncomment `en_US.UTF-8 UTF-8` in `/etc/locale.gen`
create `/etc/locale.conf` with content
```
LANG=en_US.UTF-8
```
and run `locale-gen`
### Setup hostname and hosts
echo `yourhostname` > /etc/hostname
create `/etc/hosts` with the following content:
```
127.0.0.1 localhost
::1 localhost
127.0.1.1 yourhostname.localdomain yourhostname
```
### Change root password
```
passwd root
```
### Install networkmanager
```
pacman -S networkmanager
```
```
systemctl enable networkmanager
```
### Install and setup systemd-boot
```
bootctl --path=/boot$esp install
```
Your `/boot` should look like this
```
/boot
├── EFI
│   ├── Boot
│   │   └── BOOTX64.EFI
│   └── systemd
│       └── systemd-bootx64.efi
├── initramfs-linux-fallback.img
├── initramfs-linux.img
├── loader
│   ├── entries
│   │   └── arch.conf
│   └── loader.conf
└── vmlinuz-linux
```
take note of `arch.conf` and `loader.conf`
edit/create if not existing
`/boot/loader/loader.conf`
```
default arch
timeout 4
editor 0
```

`/boot/loader/entries/arch.conf`
```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=*youruuid* rw
```
get your /dev/sda2 uuid with the command `blkid -s PARTUUID -o value /dev/sda2`(preload `:read !` if on vim)
_note: dont forget the rw at the end. I've been bitten by this a lot of times :feelsbadman:_
`exit`
`reboot`
### AND YOU SHOULD BE GOOD TO GO! ###

### References and many thanks to:
- https://ricostacruz.com/til/arch-linux-installation-summarized
- https://wiki.archlinux.org/index.php/installation_guide
