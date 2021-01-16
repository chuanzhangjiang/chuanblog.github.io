---
title: mutt安装配置文档
date: 2021-01-13 22:33:37
tags:
- mutt
- 系统配置
---

- [环境](#环境)
- [一些准备工作](#一些准备工作)
- [安装mutt-wizard](#安装mutt-wizard)
- [需要关注的目录](#需要关注的目录)
- [添加邮箱](#添加邮箱)
- [同步邮件](#同步邮件)
- [打开neomutt](#打开neomutt)
- [常用操作快捷键](#常用操作快捷键)
- [常见问题](#常见问题)
  - [同步邮件时出现异常](#同步邮件时出现异常)
  - [文件夹乱码](#文件夹乱码)
  - [不能找到文件夹问题](#不能找到文件夹问题)
  - [其他](#其他)

## 环境
ubuntu 20.04;

wm: dwm;

terminal: st;

## 一些准备工作
```
sudo apt install neomutt //客户端主程序
sudo apt install curl 
sudo apt install isync //拉取邮件工具
sudo apt install msmtp //发送邮件工具
sudo apt install lynx //简单预览html
sudo apt install abook //通讯录工具
sudo apt install urlview //使用浏览器打开邮件中的链接
sudo apt install pass //加密邮箱密码工具
```

## 安装mutt-wizard
```
git clone https://github.com/LukeSmithxyz/mutt-wizard
cd mutt-wizard
sudo make install
```
附上项目github地址：https://github.com/LukeSmithxyz/mutt-wizard

## 需要关注的目录
```
~/.mbsyncrc            #isync/mbsync配置文件
~/.config/mutt/        #mutt配置文件所在目录
~/.local/share/mail/   #isync拉取的邮件保存的目录
```

## 添加邮箱
首先在系统中生成一对密钥对：
```
gpg full-gen-key
```
然后加密存储你的邮箱密码:
```
pass init xxxxx@xx.com
```
最后开始添加邮箱账户:
```
mw -a xxxxx@xx.com
```
以上操作根据提示输入相关内容即可

## 同步邮件
执行以下命令同步邮件:
```
mbsync xxxxx@xx.com
```

## 打开neomutt
执行如下命令打开neomutt
```
neomutt
```

## 常用操作快捷键
打开neomutt之后的常用快捷键
```
m 发送邮件
j/k 下/上
ctrl-j/k 左边导航栏选择文件夹
ctrl-o 打开左边导航栏选中的文件夹
l 打开邮件，在按一次，将邮件中的内容和附件列成列表查看
h 和l相反
r 回复邮件
ctrl-b 打开邮件之后将邮件中的url列成列表然后可以用浏览器打开
i-[1-9] 进入不同的邮箱帐号
a 将选中邮件中的联系人存储到联系人列表中，发件时按tab键选择联系人
S 执行操作（如将标记为删除的文件删除）
? 帮助（其他快捷键都在这里面）
```
## 常见问题
### 同步邮件时出现异常
```
C: 0/1  B: 0/0  M: +0/0 *0/0 #0/0  S: +0/0 *0/0 #0/0
Error: flattened mailbox name '&UXZO1mWHTvZZOQ-/QQ&kK5O9ouilgU-' contains canonical hierarchy delimiter
C: 1/1  B: 0/0  M: +0/0 *0/0 #0/0  S: +0/0 *0/0 #0/0
```
打开`~/.mbsyncrc`,将该邮箱帐号对应频道的的flatten一行注释掉:
```
# flatten .
```
### 文件夹乱码
将服务端邮箱文件夹改为英文名字（部分邮箱只需要将服务端语言设置为英文即可，部分邮箱不支持修改）。

### 不能找到文件夹问题
问题截图如下:
{% asset_img cant_found_folder.png %}

问题原因：文件夹名字中存在空格

解决方案：转意空格，如下图：
{% asset_img cant_found_folder2.png %}

### 其他
可以上这里看看: https://github.com/LukeSmithxyz/mutt-wizard
