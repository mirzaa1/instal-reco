
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
mkdir /mnt/boot
```
```
mount -o uid=0,gid=0,fmask=0077,dmask=0077 /dev/nvme0n1p1 /mnt/boot
```
```
pacstrap /mnt base linux-hardened linux-firmware-intel linux-firmware-realtek mkinitcpio base-devel intel-ucode openssh ethtool rsync neovim
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
```
echo recovery > /etc/hostname
```
```
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
```
```
hwclock --systohc
```
```
nvim /etc/vconsole.conf
```
```
FONT=lat2-16
FONT_MAP=8859-2
```
```
nvim /etc/locale.gen
```
uncommenting
```
en_US.UTF-8 UTF-8
en_US ISO-8859-1
```
```
locale-gen && locale > /etc/locale.conf
```
```
sed -i 's/^LANG=C.UTF-8/LANG=en_US.UTF-8/' /etc/locale.conf
```
```
sed -i 's/^LC_ALL=/LC_ALL=en_US.UTF-8/' /etc/locale.conf
```
```
useradd -m null
```
```
passwd null
```
```
echo "null ALL=(ALL:ALL) ALL" > /etc/sudoers.d/01_null
```
```
rm /boot/initramfs-linux-*
```
```
mkdir -p /boot/efi /boot/efi/linux /boot/efi/systemd /boot/efi/rescue /boot/efi/boot
```
```
mkdir /boot/kernel
```
```
mv /boot/intel-ucode.img /boot/vmlinuz-linux-* /boot/kernel
```
```
nvim /etc/systemd/network/20-ethernet.network
```
```
[Network]
Address=[IP]/24
Gateway=10.10.1.1
DNS=1.1.1.1 8.8.8.8
MulticastDNS=yes
```
```
systemctl enable systemd-networkd.socket
```
```
systemctl enable systemd-resolved
```
```
systemctl enable sshd
```
```
mkdir /etc/cmdline.d
```
```
touch /etc/cmdline.d/{01-boot.conf,02-mods.conf,03-secs.conf,04-perf.conf,05-nets.conf,06-misc.conf}
```
```
echo "root=/dev/sda1" > /etc/cmdline.d/01-boot.conf
```
```
echo "rw" > /etc/cmdline.d/06-misc.conf
```
```
mv /etc/mkinitcpio.conf /etc/mkinitcpio.d/default.conf
```
```
nvim /etc/mkinitcpio.d/linux-hardened.preset
```

```
# mkinitcpio preset file for the 'linux-hardened' package

ALL_config="/etc/mkinitcpio.d/default.conf"
ALL_kver="/boot/kernelvmlinuz-linux-hardened"
ALL_kerneldest="/boot/kernel/vmlinuz-linux-hardened"

PRESETS=('default')
#PRESETS=('default' 'fallback')

#default_config="/etc/mkinitcpio.conf"
#default_image="/boot/initramfs-linux-lts.img"
default_uki="/boot/efi/linux/arch-linux-hardened.efi"
#default_options="--splash /usr/share/systemd/bootctl/splash-arch.bmp"

#fallback_config="/etc/mkinitcpio.conf"
#fallback_image="/boot/initramfs-linux-lts-fallback.img"
#fallback_uki="/efi/EFI/Linux/arch-linux-lts-fallback.efi"
#fallback_options="-S autodetect"
```
```
bootctl --path=/boot install
```
```
mkinitcpio -P
```
