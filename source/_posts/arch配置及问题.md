---
title: arch配置及问题
date: 2022-01-29 20:56:12
tags:
- arch
- 系统配置
---

- [安装及网络配置](#安装及网络配置)
- [桌面环境](#桌面环境)
- [安装nerd字体](#安装nerd字体)
- [安装中文字体](#安装中文字体)
- [安装中文输入法](#安装中文输入法)
- [安装chrome](#安装chrome)
- [修改`~/.zprofile`中的默认程序](#修改zprofile中的默认程序)
- [修改笔记本背关灯调节软件](#修改笔记本背关灯调节软件)
- [根据喜好修改`dwmblocks`](#根据喜好修改dwmblocks)
- [安装`wal`实现主题配色根据背景图片改变](#安装wal实现主题配色根据背景图片改变)
- [使用自己修改过的dwm](#使用自己修改过的dwm)
- [安装java](#安装java)
- [安装sbt](#安装sbt)
- [安装vscode](#安装vscode)

## 安装及网络配置
安装直接而参照[官方文档](https://wiki.archlinux.org/title/installation_guide)，bootload参考[此处](https://lapherder.tech/index.php/archives/10/)，网络配置[参考](https://lapherder.tech/index.php/archives/11/)。

## 桌面环境
直接使用luke的larbs脚本自动[配置](https://larbs.xyz/)。
```
$ curl -LO larbs.xyz/larbs.sh
$ sh larbs.sh
```

## 安装nerd字体
```
$ yay -S nerd-fonts-complete
```

## 安装中文字体
```
$ yay -S noto-fonts-cjk
```

## 安装中文输入法
首先
```
$ yay -S fcitx5-im //输入法框架
$ yay -S fcitx5-chinese-addons //中文输入法插件
```
然后，启动fcitx5-config进行配置；

接下来，配置fcitx5开机自启动，在家目录下.xprofile中加入:
```
fcitx5 -d
```

最后`~/.pam_environment`中加入如下环境变量，保证某些程序正常启动输入法:
```
GTK_IM_MODULE DEFAULT=fcitx
QT_IM_MODULE  DEFAULT=fcitx
XMODIFIERS    DEFAULT=\@im=fcitx
INPUT_METHOD  DEFAULT=fcitx
SDL_IM_MODULE DEFAULT=fcitx
GLFW_IM_MODULE DEFAULT=ibus
```

## 安装chrome
```
$ yay -S google-chrome
```

## 修改`~/.zprofile`中的默认程序

## 修改笔记本背关灯调节软件
首先替换原先使用的软件：
```
$ yay -S acpilight
```
定位`xbacklight`位置:
```
$ whic xbacklight
```
根据输出修改visudo
```
$ sudo SUDO_EDITOR=emacs visudo
```
## 根据喜好修改`dwmblocks`
修改状态栏显示内容，位置:`~/.local/src/dwmblocks/config.h`，修改完后编译安装:
```
$ sudo make clean install
```
## 安装`wal`实现主题配色根据背景图片改变
```
$ yay -S wal
```
**note: 修改背景图片，使用lf选中对应图片，按b。**

## 使用自己修改过的dwm
仓库地址:git@github.com:chuanzhangjiang/dwm-luke.git

## 安装java

## 安装sbt

## 安装vscode