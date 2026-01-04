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
