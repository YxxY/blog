---
title: Linux从入门到放弃
date: 2017-06-20 20:35:38
categories:
- Linux
tags: 
- Linux
- Ubutun
---

很早就有学linux的想法，曾经也尝试过，特地装了双系统，但可能打开的姿势和时机不对，没过多久就不了了之了。如今已经工作一段时间了，感觉这是一项必须掌握的技能了。本着talk is cheap， just do it的实用主义原则，仅以使用目标为导向，逐步先达到我操作windows的水平，其他以后再说，捞着芝麻算芝麻，西瓜如果抱不动就不要了。以下以ubutun16.04为实验机，全程百度必应，行文仅为学习的总结及备忘录，内容逐步补充，内容无增加之时，则实现点题。
<!--more-->

## 认识目录
	windows下目录就是C盘D盘之类，很好理解，系统盘和非系统盘。
那linux下呢？盗图一张如下：
![dir](http://opxo4bto2.bkt.clouddn.com/linux_dir.png)
意思就是说只有一块盘，相当于windows下只有一个C盘。
根目录就是`/`，相当于windows下的`C:\`。该目录下`/etc`存放系统各种配置文件，`/root`为管理员目录，非管理员用户目录都在`/home`下等等。
## 执行命令
	windows下win+R 打开命令行时，显示当前用户目录路径
但linux下win+alt+T打开terminal时，显示的符号是啥意思呢？
普通用户登录会显示`~$`，root用户登录显示`~#`。
非root（管理员）用户的符号会显示$，而root用户则会显示#（代表权力至高无上）。
`~`代表当前用户目录，root用户则是`/root/`，非root则是`/home/name/`。
`-`代表之前操作的目录，非常有用。

## 文件编辑
	windows下编辑文件，右击选择文件打开方式，编辑，ctrl+S保存，关闭文件。
linux下键盘操作是主力。以下总结vi编辑器的基本使用。当然可以用gedit等GUI编辑器，应该就没有学习成本了。
### 打开|新建文件
    可以右键打开或新建，和windows下一样，如果是命令模式，推荐使用vi
### vi使用
`vi filename` 不存在该文件则新建

	vi filename	（此时光标在第一行首）
	vi + filename （光标在最后一行首，输入时注意加号两边空格）
	vi +/pattern filename （匹配模式，光标置于匹配的第一个字段处，无匹配项会提示）

进入文件后此时是命令模式，输入字符不会显示。
在命令模式下输入插入命令i（insert）、附加命令a （append）、打开命令o（open）、修改命令c（change）、取代命令r或替换命令s都可以进入文本输入模式，此时编辑器下方会显示`-- INSERT --`样式,如无则需设置`set showmode`, 同理设置行号`set nu`
在命令模式下敲【: / ?】中任一个可进入末行模式，执行完命令回到命令模式，或敲ESC退出到命令模式。
命令模式和末行模式我个人觉得没有明确界限，大概就是末行模式可以在屏幕上看到命令符吧
vi的操作的命令需熟练掌握，自行查找。

### 删除文件
`rm -fir file|dir`
`rm file1 file2` 删除多个文件

    -f 强制删除，忽略不存在的文件
    -i 删除前确认
    -r 递归删除文件目录

### 复制剪切文件
`cp src dest` cp命令可自行了解
`cat file1 > file2` 也能达到复制文件的目的
`mv fp1 fp2` 剪切意味着移动move，也可以起到更名的作用
针对mv说明：

    -b ：若需覆盖文件，则覆盖前先行备份。备份文件名为`filename~` 
    -f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖
    -i ：若目标文件 (destination) 已经存在时，就会询问是否覆盖！
    -u ：若目标文件已经存在，且 source 比较新，才会更新(update)
    -t ：--target-directory=DIRECTORY 即指定mv的目标目录，
        该选项适用于移动多个源文件到一个目录的情况，此时目标目录在前，源文件在后。
mv -u file1 file2

### 合并文件
`cat file1 file2 > file` 
当`file`不存在会新建，若存在已有内容会被覆盖
参数：

    -n 或 --number 由 1 开始对所有输出的行数编号
    -b 或 --number-nonblank 和 -n 相似，只不过对于空白行不编号
    -s 或 --squeeze-blank 当遇到有连续两行以上的空白行，就代换为一行的空白行
    -v 或 --show-nonprinting 展现出非打印字符

eg：把 file1 和 file2的文件内容加上行号后依次输入 file3 文件
    cat -n file1 file2 > file3

### 查找文件
可以使用系统自带的搜索功能，输入关键字即可，另外也可使用find命令
`find /home -name "*abc*"`
`locate filename` 同样支持匹配

### 文件权限
`ls -l filename|dirname` 查看权限
权限分为读、写、执行。权限针对的三组人员，文件拥有者、用户群组、其他人
更改权限使用`chmod `命令，使用时 `chmod --help`查看说明
记住其中一种使用方法就好 `chmod abc file`,其中a,b,c各为一个数字，分别表示User、Group、及Other的权限。

    r=4，w=2，x=1 
chmod a=rwx file 和 chmod 777 file 效果相同
修改目录下所有文件权限则使用`chmod -R 777 * `

修改文件拥有者和群组，使用 `chown`命令，普通用户不能将自己的文件改变成其他的拥有者。其操作权限一般为管理员
`chown [option] [new_owner]:[new_group] file`

    -c 显示更改的部分的信息
    -f 忽略错误信息
    -h 修复符号链接
    -R 处理指定目录以及其子目录下的所有文件
    -v 显示详细的处理信息
eg: `chown -R -v root:admin test `将test目录下所有文件拥有者改为root，群组改为admin

## windows远程linux
命令行远程当然是用SSH，客户端很多，推荐Xshell
GUI远程 也有很多选择, 例如vnc、nomachine，这里说下另一种：
xRDP + Xfce 实现Windows远程桌面连接

1. 安装xRDP及vncserver
sudo apt-get install xrdp 
sudo apt-get install vnc4server tightvncserver
2. 安装Xfce桌面环境 
sudo apt-get install xubuntu-desktop
3. 设置xRDP 
echo xfce4-session >~/.xsession
4. 设置配置文件 
sudo gedit /etc/xrdp/startwm.sh 
在. /etc/X11/Xsession 前一行插入 xfce4-session
5. 重启 xrdp 
sudo service xrdp restart
6. windows通过mstsc连接

Xface桌面tab不能自动补全的问题
settting->window manager->Keyboard，清除tab快捷键占用

## 查杀进程
    `ps -e ` 查看所有进程，慢慢找吧……
    `ps aux|grep <key>` 通过关键字查找目标进程
    `sudo kill [code] <PID>` PID 为进程id，这个和windows下差不多
    code默认为`-9`,表示发送kill信号，另外提两个
    `-STOP` 停止进程，但不结束
    `-CONT` 继续运行已停止的进程



## shell脚本
对应windows的批处理
若执行失败时，尝试sudo执行

未放弃，待续！



