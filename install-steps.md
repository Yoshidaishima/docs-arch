# Arch install steps for simple quick VM
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
mkfs.fat -F32 $P1_PATH
mkfs.ext4 $P2_PATH
```

### Mount partitions

```
mount $P1_PATH /mnt
mount -m $P2_PATH /mnt/efi
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

