
```
mkfs.vfat -F32 -S 4096 /dev/nvme0n1p1
```
```
mkfs.ext4 -b 4096 /dev/sda1
```
```
mount /dev/sda1 /mnt
```
```
pacstrap /mnt base linux-hardened linux-firmware-intel linux-firmware-realtek mkinitcpio base-devel intel-ucode openssh ethtool rsync
```
```
genfstab -U /mnt >> /mnt/etc/fstab
```
```
cp /etc/systemd/network/20-ethernet.network /mnt/etc/systemd/network/
```
```
arch-chroot /mnt
```
