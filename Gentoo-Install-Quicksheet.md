1. Download [LiveGUI](https://www.gentoo.org/downloads/) USB image, flash, boot.
2. Configure the network using `nmtui`.
3. Prepare the disk.
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
4. Download [stage3 file](https://www.gentoo.org/downloads/) and copy into disk.
  ```
  tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner -C /mnt/gentoo
  ```
5. Use my own portage config for my laptop.
  ```
  git clone https://github.com/Zirui-Ding/gentoo-config
  cp gentoo-config/portage/make.conf /mnt/gentoo/etc/portage/make.conf
  rm -rf /mnt/gentoo/etc/portage/package.use
  cp gentoo-config/portage/package.use /mnt/gentoo/etc/portage/package.use
  cp /etc/resolv.conf /mnt/gentoo/etc/resolv.conf
  ```
6. Chroot
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
  passwd
  ```
7. Install repository and update packages with "~amd64" enabled.
  ```
  emerge-webrsync
  emerge dev-vcs/git app-eselect/eselect-repository
  rm -rf /var/db/repos/gentoo
  eselect repository list
  eselect repository enable gentoo guru science steam-overlay
  emerge --sync
  emerge -avuDN @world
  ```
8. Config system
  ```
  lsblk
  blkid | grep /dev/nvmeX
  ```
Copy the output to [chatgpt](https://chatgpt.com) and let it generate an `/etc/fstab` file with partuuid.
  ```
  echo gentoo > /etc/hostname
  nano /etc/hosts
  systemd-machine-id-setup
  systemd-firstboot --prompt
  systemctl preset-all --preset-mode=enable-only
  ```
9. Install networkmanager and kernel sources
  ```
  emerge gentoo-sources efibootmgr nwtworkmanager vim neovim dev-vsc/git
  ```
10. Compile kernel
  ```
  git clone https://github.com/Zirui-Ding/gentoo-config
  eselect kernel list
  eselect kernel set 1
  cp kernel-config /usr/src/linux/.config
  cd /usr/src/linux/
  make menuconfig
  make -j24 
  ```
11. Install kernel and generate boot entry
  ```
  efibootmgr -c -d /dev/nvmeXn1 -p 1 -L "Gentoo Linux" -l '\vmlinuz'
  ```
12. Reboot to new OS
   ```
   reboot
   ```
13. To be continue...
