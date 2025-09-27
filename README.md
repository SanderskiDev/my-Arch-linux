# my-Arch-linux

in this repo i will post my journey with Arch linux from private documentation i made so far
my intentions are to make installation guide that would have no issue guarranteed further more in close future i will be going to make script like a Archinstall but with my touch
Hope you will find my documentation helpful!
the documentation goes as follows:

---

**1- timedatectl set-ntp true** #update system clock its more of recomended

**2- partition disks fdisk or cfdisk**

**3- format:**

```bash
ESP: mkfs.fat -F32 /dev/sda1
Root: mkfs.ext4 /dev/sda2
Swap: mkswap /dev/sda3
```

**4- Mount the File Systems:**

```bash
Create ESP mount point: mkdir -p /mnt/boot/efi
Mount ESP: mount /dev/sda1 /mnt/boot
Mount root: mount /dev/sda2 /mnt
Enable swap: swapon /dev/sda3
```

**5- install:**

```bash
pacstrap -K /mnt base linux linux-firmware
```

**6- Generate fstab:**

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

**7- Chroot into the new system:**

```bash
arch-chroot /mnt
```

**8- Time Zone (IN CHROOT):**

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

**9- locale stuff (IN CHROOT):**

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
sed -i '/en_US.UTF-8/s/^#//g' /etc/locale.gen
sed -i '/pl_PL.UTF-8/s/^#//g' /etc/locale.gen
locale-gen
```

**10- hostname (IN CHROOT):**

```bash
echo "archlinuxname" >> /etc/hostname
```

**11- (optional?) hosts (IN CHROOT):** #you can add your rotuer there

```bash
echo "192.168.0.1    router" >> /etc/hosts
# /\ rotuer ip
```

**'script': (its very not recomended to use that script)**

```bash
gateway_ip=$(ip route show default | awk '/default/ {print $3}')
echo "$gateway_ip    router" >> /mnt/etc/hosts
```

**12- use passwd for root (IN CHROOT) #so there wouldnt be any confusion**

```bash
passwd
```

**13- boot stuff (IN CHROOT)**

```bash
bootctl install
```

**14 PATH A: Using systemd-boot- install nano and create a boot entry file in /boot/loader/entries/arch.conf (IN CHROOT)**

```bash
pacman -S nano
blkid -s PARTUUID -o value /dev/sda2 #in root get PARTUUID
nano /boot/loader/entries/arch.conf
```

#type this in arch.conf
```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=your_root_partition_partuuid rw
```

EXAMPLE:
```
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=1811f51e-da89-49dc-b6d9-6959ae01c391 rw
```

**14 PATH B: Using GRUB- Install the GRUB packages and other stuff:** #its more recomended if you dont care about the speed of booting up

```bash
pacman -S grub efibootmgr
mkinitcpio -P
# (on efi):
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

**15- Reboot**

```bash
Exit the chroot: exit
Unmount: umount -R /mnt
Reboot: reboot
```

---

(its advaiced to create Sudo user instead of abusing root that often)

**Creating a User and Setting up Sudo**

```bash
pacman -S sudo
useradd -m -G wheel -s /bin/bash yourusername
passwd yourusername
EDITOR=nano visudo
%wheel ALL=(ALL:ALL) ALL
```

---

## CHAPTER 2 "Internet Config"

#assuming youre working on "Class A"

**configure your internet by via using ip command:**

```bash
ip addr add 192.168.0.*/24 dev enp0s25 #ip addr show
sudo ip route add default via 192.168.0.1 dev enp0s25 #ip route show
echo "nameserver 8.8.8.8" >> /etc/resolv.conf
echo "nameserver 8.8.4.4" >> /etc/resolv.conf
echo "nameserver 1.1.1.1" >> /etc/resolv.conf
echo "nameserver 1.0.0.1" >> /etc/resolv.conf
```

OR (recomended) put your router/gateway there

```bash
echo "nameserver 192.168.0.1" >> /etc/resolv.conf
```

**GET DHCP, FIREWALL WIRELESS CONNETIONS:**

```bash
pacman -S networkmanager
systemctl enable --now NetworkManager
Find your connection names: nmcli connection show
pacman -S ufw
nano /etc/ufw/sysctl.conf
```

Add to `/etc/ufw/sysctl.conf`:
```
net/ipv4/ip_forward=1
net/ipv6/conf/default/forwarding=1
net/ipv6/conf/all/forwarding=1
```

#FOR SHARING STUFF FOR EXAMPLE INTERNET TO YOUR PHONE, IF YOU DO THUS YOU MUST PREPARE beofre.rules file (not must have)

```bash
nano /etc/ufw/before.rules
```

Add to `/etc/ufw/before.rules`:
```
*nat
:POSTROUTING ACCEPT [0:0]
-A POSTROUTING -o wlan0 -j MASQUERADE
COMMIT
```

**HOSTSPOT**

```bash
nmcli device wifi hotspot ssid "MyHotspot" password "im the student of highschool :P" ifname wlan0
systemctl enable --now ufw OR ufw enable
ufw status verbose
```

**WIRELESS INTERNET:**

```bash
pacman -S iwd
sudo systemctl enable iwd
sudo systemctl start iwd
iwctl
station wlan0 scan
station wlan0 get-networks
station wlan0 connect "Your Network Name"
#your internet should work by now
```
