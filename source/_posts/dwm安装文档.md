---
title: dwm安装文档
date: 2021-01-13 21:48:37
tags: 
- dwm
- 系统配置
---
- [前言](#前言)
- [关于DWM](#关于dwm)
- [准备工作](#准备工作)
  - [基础依赖](#基础依赖)
  - [后期功能性软件](#后期功能性软件)
- [DWM安装](#dwm安装)
- [设置DWM启动方式](#设置dwm启动方式)
  - [使用display manager启动](#使用display-manager启动)
  - [使用startx命令从文字界面启动(推荐)](#使用startx命令从文字界面启动推荐)
- [为软件指定打开标签](#为软件指定打开标签)
- [可能遇到的问题](#可能遇到的问题)
  - [状态栏无法显示emoji](#状态栏无法显示emoji)

## 前言
本教程直接使用本人配置好的DWM，建议新手直接使用我的配置，等上手之后在在行下载官方源码进行编译安装配置。
## 关于DWM
可以看B站CW大佬的[视频](https://www.bilibili.com/video/BV11J411t7RY)
## 准备工作
### 基础依赖
```
$ sudo apt-get install suckless-tools libx11-dev libxft-dev libxinerama-dev gcc make
```
### 后期功能性软件
透明配置支持
```
$ sudo apt install compton

或者用下面的工具

$ sudo apt install xcompmgr
```

背景图片设置工具
```
$ sudo apt install feh
```
电源监控工具
```
$ sudo apt install acpi acpitool
```
背光灯调整工具
```
$ sudo apt install light
```
为背光灯调整工具设置sudo免密码
```
$ sudo visudo
```
然后在文本最后加入如下代码
```
{登录系统的用户名} ALL=NOPASSWD:/usr/bin/light
```
安装截图工具
```
$ sudo apt install flameshot
```
安装数字键盘工具
```
$ sudo apt install numlockx
```
虚拟机可能需要以下软件

virtualbox
```
$ sudo apt-get install virtualbox-guest-utils virtualbox-guest-X11
```
vmware
```
$ sudo apt-get install open-vm-tools open-vm-desktop
```
## DWM安装

获取源码
```
$ git clone https://github.com/chuanzhangjiang/dwm.git
```
移动到源码目录修改config.h文件，自行修改下图中用户目录名称

{% asset_img config_modify.png %}

获取dwm自定义脚本
```
$ git clone https://github.com/chuanzhangjiang/dwm-script.git
```
将脚本文件软链接到`～/.dwm`目录
```
$ ln -s  ~/dwm-script的目录 ~/.dwm
```
移动到dwm源码目录执行安装命令
```
$ sudo make clean install
```
## 设置DWM启动方式
有两种启动方式可以选择
### 使用display manager启动
以ubuntu 20.04为例,ubuntu 20.04使用gdm3做为display manager，配置完成之后可以在登录界面选择dwm作为桌面启动，如下图：

{% asset_img desktop_entry.png %}

具体配置方式，进入`/usr/share/xsessions/`目录，新建文件`dwm.desktop`,输入内容：
```
[Desktop Entry]
Encoding=UTF-8
Name=Dwm
Comment=Dynamic window manager
Exec=dwm
Icon=dwm
Type=XSession
```
### 使用startx命令从文字界面启动(推荐)
此方式开机更加快速，使用更加灵活，系统资源占用更少。

**首先将系统改为默认进入文字界面**


修改grub配置,打开文件`/etc/default/grub`,将`GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"`的改为`GRUB_CMDLINE_LINUX_DEFAULT="text"`然后执行命令
```
$ sudo update-grub
```
将启动等级改为多用户等级，执行如下命令：
```
$ systemctl set-default multi-user.target 
// 如果想改回启动图形界面执行下面
$ systemctl set-default graphical.target
```
最后修改`~/.xsession`文件（如果没有就新建），在最后一行加入
```
exec dwm
```
重启电脑，执行startx命令，直接进入dwm，同时也可以执行`sudo systemctl start gdm.service`命令，打开gdm3的用户登录界面。

## 为软件指定打开标签
```
xprop | grep WM_CLASS
```
鼠标会变为十字架,用十字架点击想要被指定的软件的打开窗口，terminal就会显示该软件的instance和class:
```
WM_CLASS(STRING) = instance, class
```
将信息填入config.h的下列位置:

{% asset_img rule.png %}

`tags mask`用于指定打开的表情序号，一号标签为二进制`000000001`,二号为`000000010`,类推。

重新编译安装即可。
完。

## 可能遇到的问题
### 状态栏无法显示emoji
**问题说明**
问题是由于dwm和st以来的xft出现bug导致的，arch平台可以安装`libxft-bgra`补丁来修复xft的bug，其他平台解决此问题不太容易，只能等libxft更新。
本文基于ubuntu 20.04,采用迂回战术解决此问题。
**安装emoji支持字体**
```
sudo apt install fonts-symbola
```
**配置dwm**
修改dwm的`config.h`文件：

{% asset_img dwm-symbola.png %}

**重新编译安装**
重新编译安装dwm即可。