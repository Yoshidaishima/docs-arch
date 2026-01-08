# Arch install steps
### Tips 
- **Quicker setup** : Use archinstall command to bootstrap the system
  - **Note** : Cannot fine tune partitioning and swap allocation
- **Useful tools** : dpaste.com, allows copy paste to remote terminal based environments from host

## System setup

### Pacman
```
pacman -Syu
```
May require boot param to increase working space
- during systemd-boot screen press **e**
- append the following to the params to increase the overlay to 1 Gig
```
cow_spacesize=1G
```
- continue to boot
### Check disks

```
lsblk
fdisk -l
```

### Partition
gdisk needed for drives above 2TB
```
fdisk /dev/$DEVICE
```

> partition 1 = 512M efi
> partition 2 = 32G swap
> partition 3 = root
> partition 4 = home (for a personal machine, not required)


### Format

```
mkfs.fat -F32 $EFI_PARTITON
mkswap $SWAP_PARTITION
swapon $SWAP_PARTITION
mkfs.ext4 $ROOT_PARTITION
```

### Mount partitions

```
mount $ROOT_PARTITION /mnt
mount -m $EFI_PARTITON /mnt/efi
```

### Strap

```
pacstrap /mnt base linux linux-firmware
```

### Gen fstab

```
genfstab -U /mnt >> /mnt/etc/fstab
```

### Change root

```
arch-chroot /mnt
```

### Set locale and hostname

```
sed -r -i 's/#(en_US.UTF-8)/\1/' /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" >> /etc/locale.conf

echo $NEW_HOST_NAME >> /etc/hostname
```

### Gen initial RAM disk
> Pacstrap should have already executed this step
> :. likely not needed

```
mkinitcpio -P
# on /etc/vconsole.conf error
touch /etc/vconsole.conf
```

### Set root pass

```
passwd
```

### Bootloader

```
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/efi --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### Create user

```
useradd -m -G wheel $NEW_USER_NAME
passwd $NEW_USER_NAME
```

## Install packages
### Ethernet
```
pacman -S sudo openssh iwd dhcpcd git curl vi less jq ranger tmux
```
### Enable
```
systemctl enable dhcpcd
systemctl enable iwd
systemctl enable sshd
```
### Sudo setup
```
visudo
# uncomment wheel group
```

## Reboot
> Backout of chrooted env
```
umount /mnt/efi
umount /mnt
reboot now
```

## Update mirrors
```
pacman -S reflector
# backup old list
sudo cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
sudo reflector --list-countries
sudo reflector --country "United Kingdom" \
--protocol https \
--age 12 --latest 20 \
--sort rate \
--save /etc/pacman.d/mirrorlist
```

## Wifi setup
Connect
```
iwctl
station list
station $INTERFACE scan
station $INTERFACE connect $SSID
station $INTERFACE show
```
To set up static ip add to the following files
```
# /etc/iwd/main.conf
[General]
EnableNetworkConfiguration=true
[Settings]
AutoConnect=true

# /var/lib/iwd/<SSID>
[IPv4]
Address=<Static IP>
Netmask=<Netmask>
Gateway=<Gateway>
Broadcast=<Broadcast>
DNS=<DNS>
```

## Mac Specific
This relates specifically to legacy apple hardware (circa 2013) using proprietary broadcom drivers after a fresh arch install

Should be done after first boot

Download
```
pacman -S broadcom-wl-dkms
```
Install headers
```
sudo pacman -S $(pacman -Qsq "^linux" | grep "^linux[0-9]*[-rt]*$" | awk '{print $1"-headers"}' ORS=' ')
```
Remove conflicting modules
```
rmmod b43 b43legacy ssb bcm43xx brcm80211 brcmfmac brcmsmac bcma wl
```
Load wl
```
modprobe wl
```
If modprobe fails, need to remove old dkms modules
```
# list modules
dkms status
dkms remove broadcom-wl/6.30.223.271 -k $MISSING_KERNEL
```
Rebuild and install dkms module
```
dkms build -m broadcom-wl -v $VER -k $(uname -k) --force
dkms install -m broadcom-wl -v $VER -k $(uname -k) --force
```
**Note** the version must match the downloaded broadcom-wl-dkms, check with ```pacman -Q broadcom-wl-dkms```

Confirm with ```pacman -Q broadcom-wl-dkms```

Reboot and check ```ip link```

# Further Setup
Use ansible to further configure the remote hosts once the base system is installed and accessable through ssh

Allows to replicable steps and reduces the odds of generating snowflakes
