# Arch install steps for simple quick VM
### Check disks

``` bash
lsblk
fdisk -l
```

### Partition

``` bash
fdisk
```

> partition 1 = 512M for efi  
> partition 2 = Remaining drive as root

### Format

``` bash
mkfs.fat -F32 $P1_PATH
mkfs.ext4 $P2_PATH
```
