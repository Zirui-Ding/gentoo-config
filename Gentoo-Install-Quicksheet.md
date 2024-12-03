1. Download LiveGUI USB image, flash, boot.

2. Configure the network using `nmtui`.

3.
```
lsblk
cfdisk /dev/nvmeXn1
...
mkfs.vfat -F 32 /dev/nvmeXn1p1
mkswap -L SWAP /dev/nvmeXn1p2
mkfs.xfs -L ROOT /dev/nvmeXn1p2

mount /dev/nvmeXn1p3 /mnt/gentoo
mkdir /mnt/gentoo/boot
mount /dev/nvmeXn1p1 /mnt/gentoo/boot  # I do not use /efi and "installkernel"
```

4. Download stage3 file. [Gentoo Downloads](https://www.gentoo.org/downloads/)

5.
```
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo
```

6.
```
git clone https://github.com/Zirui-Ding/gentoo-config
cp gentoo-config/portage/make.conf /mnt/gentoo/etc/portage/make.conf
rm -rf /mnt/gentoo/etc/portage/package.use
cp gentoo-config/portage/package.use /mnt/gentoo/etc/portage/package.use
```

7.
```
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
mount --make-slave /mnt/gentoo/run

chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```
