```
nvme format /dev/nvme0n1 --ses=1 
```
```
parted /dev/nvme0n1 mklabel gpt
```
```
sgdisk --zap-all /dev/sda
```
```
parted /dev/sda mklabel gpt
```
```
pvcreate --dataalignment 4096 /dev/sda
```
```
vgcreate cave /dev/mapper/cave
```
```
lvcreate -L 5G cave -n reco
```
```
lvcreate -l50%FREE cave -n pool
```
```
sudo sfdisk /dev/nvme0n1 << EOF
label: gpt
unit: sectors
first-lba: 2048
grain: 4096

# Partition 1: 2.5G (Aligned to 4K)
size=2.5G, type=linux
# Partition 2: 512MB (Aligned to 4K)
size=512M, type=linux
# Partition 3: 50G (Aligned to 4K)
size=50G, type=linux
# Partition 4: Remainder (Aligned to 4K)
type=linux
EOF
```
```
mkfs.fat -F 32 -S 4096 -n BOOT /dev/nvme0n1p1
```
```
sudo mkfs.xfs -s size=4096 /dev/cave/reco
```

```
mount /dev/cave/reco /mnt
```
```
mkdir /mnt/boot
```
```
mount -o uid=0,gid=0,fmask=0077,dmask=0077 /dev/nvme0n1p1 /mnt/boot
```
```
pacstrap /mnt base linux-hardened linux-hardened-headers linux-firmware-intel linux-firmware-realtek linux-firmware-other mkinitcpio base-devel intel-ucode openssh ethtool rsync neovim firewalld bubblewrap-suid less polkit git lvm2 xfsprogs
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
echo [nama hostname] > /etc/hostname
```
```
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
```
```
hwclock --systohc
```
```
mkdir /etc/systemd/timesyncd.conf.d
```
```
nvim /etc/systemd/timesyncd.conf.d/local.conf
```
```
[Time]
NTP=0.id.pool.ntp.org 1.id.pool.ntp.org 2.id.pool.ntp.org 3.id.pool.ntp.org
FallbackNTP=time.cloudflare.com time.google.com time.aws.com
```
```
timedatectl set-ntp true
```
```
timedatectl set-timezone Asia/Jakarta
```
```
timedatectl status
```
```
timedatectl show-timesync --all
```
```
systemctl enable systemd-timesyncd.service
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
useradd -m http
```
```
echo "http ALL=(ALL:ALL) ALL" >> /etc/sudoers
```
```
vim /etc/passwd
```
> change line http:x:33:33::/srv/http:/usr/bin/nologin
```
root:x:0:0::/root:/usr/bin/bash
bin:x:1:1::/:/usr/bin/nologin
daemon:x:2:2::/:/usr/bin/nologin
mail:x:8:12::/var/spool/mail:/usr/bin/nologin
ftp:x:14:11::/srv/ftp:/usr/bin/nologin
http:x:33:33::/srv/http:/usr/bin/nologin
```
```
http:x:33:33::/srv/http:/usr/bin/bash
```
```
passwd http
```
```
chage -E -1 http 
```
```
su http
```
```
sudo su
```
```
exit
```
```
exit
```
```
echo '' > /usr/lib/os-release
```
```
vim /usr/lib/os-release
```
```
NAME="Blackbird"
PRETTY_NAME="Blackbird"
ID=blackbird
BUILD_ID=rolling
ANSI_COLOR="38;2;23;147;209"
HOME_URL="https://blackbird.lektor.co.id/"
DOCUMENTATION_URL="https://blackbird.lektor.co.id/"
SUPPORT_URL="https://blackbird.lektor.co.id/support/"
BUG_REPORT_URL="https://gitlab.blackbird.org/groups/issues"
PRIVACY_POLICY_URL="https://blackbird.lektor.co.id/privacy-policy/"
LOGO=blackbird-logo
```
```
vim /etc/pacman.conf
```
> add TrustedOnly on line SigLevel = Required DatabaseOptional
```
SigLevel = Required DatabaseOptional TrustedOnly
```
uncomment
```
UseSysLog
Color
VerbosePkgLists
```
## sshd
```
systemctl enable sshd
```

## loging config
```
mkdir -p /etc/systemd/journald.conf.d/
```
```
vim /etc/systemd/journald.conf.d/01-default.conf
```
```
[Journal]
SystemMaxUse=1G
SystemKeepFree=500M
RuntimeMaxUse=200M
RuntimeKeepFree=50M
MaxFileSec=1month
Storage=persistent
```

## sleep config
```
mkdir -p /etc/systemd/sleep.conf.d/
```
```
vim /etc/systemd/sleep.conf.d/01-blackbird.conf
```
```
[Sleep]
AllowSuspend=no
AllowHibernation=no
AllowHybridSleep=no
AllowSuspendThenHibernate=no
```

## coredump config
```
vim /etc/systemd/coredump.conf
```
comment "[coredump]" adding on last line
```
[Coredump]
Storage=none
ProcessSizeMax=0
```

## login sudoers
```
vim /etc/sudo.conf
```
adding on last line
```
## Config Log
Defaults logfile="/var/log/sudo.log"
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
systemctl enable firewalld
```

```
rm /boot/initramfs-linux-*
```
```
mkdir -p /boot/efi /boot/efi/linux /boot/efi/systemd /boot/efi/rescue /boot/efi/boot /boot/kernel
```

```
mv /boot/intel-ucode.img /boot/vmlinuz-linux-* /boot/kernel
```

```
mkdir /etc/cmdline.d
```
```
touch /etc/cmdline.d/{01-boot.conf,02-mods.conf,03-secs.conf,04-perf.conf,05-nets.conf,06-misc.conf}
```
```
echo "root=/dev/cave/reco" > /etc/cmdline.d/01-boot.conf
```
```
echo "ipv6.disable=1" > /etc/cmdline.d/04-perf.conf
```
```
echo "rw quiet" > /etc/cmdline.d/06-misc.conf
```

```
rm -fr /etc/mkinitcpio.conf.d
```
```
mv /etc/mkinitcpio.conf /etc/mkinitcpio.d/default.conf
```
```
vim /etc/mkinitcpio.d/default.conf
```
> masukin lvm2 di hooks

```
nvim /etc/udev/rules.d/81-wol.rules
```
```
ACTION=="add", SUBSYSTEM=="net", NAME=="en*", RUN+="/usr/bin/ethtool -s $name wol g"
```
```
ethtool interface | grep Wake-on
```

```
mv /etc/mkinitcpio.conf /etc/mkinitcpio.d/default.conf
```
```
echo "" > /etc/mkinitcpio.d/linux-hardened.preset
```
```
nvim /etc/mkinitcpio.d/linux-hardened.preset
```

```
# mkinitcpio preset file for the 'linux-hardened' package

ALL_config="/etc/mkinitcpio.d/default.conf"
ALL_kver="/boot/kernel/vmlinuz-linux-hardened"
ALL_kerneldest="/boot/kernel/vmlinuz-linux-hardened"

PRESETS=('default')
#PRESETS=('default' 'fallback')

#default_config="/etc/mkinitcpio.conf"
#default_image="/boot/initramfs-linux-lts.img"
default_uki="/boot/efi/linux/blackbird-recovery.efi"
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
```
exit
```
```
umount -R /mnt
```

