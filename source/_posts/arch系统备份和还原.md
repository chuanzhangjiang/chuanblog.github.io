---
title: arch系统备份和还原
date: 2022-01-30 20:25:42
tags:
- arch
- 备份还原
---

## 前言
只备份根分区, home分区直接将重要文件拷贝下来即可。

## 安装pigz
```
$ pacman -S pigz
```

## 清理缓存
```
$ pacman -Sc
```

## 在`~/.config`目录下创建不备份目录列表
```
$ emacs ~/.config/exclude
```
加入如下内容
```
/proc/*
/dev/*
/sys/*
/tmp/*
/mnt/*
/media/*
/run/*
/var/lock/*
/var/run/*
/var/lib/pacman/*
/var/cache/pacman/pkg/*
/lost+found
```

## 备份命令
将根分区备份到home分区中的`backup/root-backup.tgz`中：
```
$ sudo tar --use-compress-program=pigz -cvpf ~/backup/root-backup.tgz --exclude-from=/home/chuan/.config/exclude /
```

## 使用liveCD进入系统

## 格式化根分区和boot分区
```
# mkfs.ext4 /dev/root_partition
# mkfs.fat -F 32 /dev/efi_system_partition
```

## 挂载根分区和home分区
```
# mkdir /mnt/root
# mkdir /mnt/home
# mount /dev/root_partition /mnt/root
# mount /dev/home_partition /mnt/home
```

## 在liveCD中安装pigz

## 解压home分区下的根分区备份文件到根分区中
```
# tar --use-compress-program=pigz -xvpf /mnt/home/chuan/backup/root-backup.tgz -C /mnt/root/
```

## 取消挂载
```
# umount /mnt/root
# umount /mnt/home
```

## 删除root目录
```
# rm -r /mnt/root
```

## 重新按照linux结构挂载
```
# mount /dev/root_partition /mnt
# mount /dev/home_partition /home
# mount /dev/efi_partition /boot
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