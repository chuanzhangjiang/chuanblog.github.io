---
title: arch系统备份和还原
date: 2022-01-30 20:25:42
tags:
- arch
- 备份还原
---

## 前言
只备份根分区和boot分区,home分区直接将重要文件拷贝下来即可。

## 安装pigz
```
$ pacman -S pigz
```

## 清理缓存
```
$ pacman -Sc
```

## 创建不备份目录列表
```
$ emacs ~/backup/exclude
```
加入如下内容
```
/home/*
/proc/*
/dev/*
/sys/*
/tmp/*
/mnt/*
/media/*
/run/*
/var/lock/*
/var/run/*
/var/cache/pacman/pkg/*
/lost+found
```

## 备份命令
进入`backup`目录,执行备份：
```
$ cd backup
$ sudo tar --use-compress-program=pigz -cvpf backup.tgz --exclude-from=exclude /
```

## 使用liveCD进入系统

## 格式化根分区和boot分区
```
# mkfs.ext4 /dev/root_partition
# mkfs.fat -F 32 /dev/efi_system_partition
```

## 挂载分区
```
# mount /dev/root_partition /mnt
# mkdir /mnt/home
# mount /dev/home_partition /mnt/home
# mkdir /mnt/boot
# mount /dev/boot_partition /mnt/boot
```

## 解压home分区下的根分区备份文件到根分区中
```
# cd /mnt/home/<username>/backup
# tar --use-compress-program=pigz -xvpf backup.tgz -C /mnt
```

## 开启swap
```
# swapon /dev/swap_partition
```

## 重新生成fstab
```
# genfstab -U /mnt > /mnt/etc/fstab
```

## chroot
```
# arch-chroot /mnt
```

## 重新配置bootloader
```
# grub-mkconfig -o /boot/grub/grub.cfg
```

## 重启进入系统