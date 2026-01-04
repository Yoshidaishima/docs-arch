# Arch install steps for simple quick VM
## System setup
### Check disks

```
lsblk
fdisk -l
```

### Partition

```
fdisk
```

> partition 1 = 512M for efi  
> partition 2 = Remaining drive as root

### Format

```
mkfs.fat -F32 $EFI_PARTITON
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
pacman -S dhcpcd
systemctl enable dhcpcd
```

### Wifi

```
pacman -S iwd
systemctl enable iwd
```

### Other

```
pacman -S vi sudo
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
