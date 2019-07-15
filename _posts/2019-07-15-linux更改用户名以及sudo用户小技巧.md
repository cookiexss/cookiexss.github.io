---
layout:     post
title:      linux更改用户名以及sudo用户小技巧
subtitle:   修改linux用户名并修改用户目录名以及sudo用户小技巧
date:       2019-07-15
author:     LISTONE
header-img: img/post-bg-digital-native.jpg
catalog: true
tags:
    - linux
    - Notes
---
>本文首发于博主公众号LISTONE，欢迎关注哦！

## 前言
&emsp;&emsp;**一、** 历时几天，终于下完了一个3.4G大小的一个ubuntu虚拟机压缩包（百度云限速，本人穷，买不起会员。。。），这个虚拟机是前人搭建好的一个pwn做题环境，它的默认名字是他平时的代号，这个就让我很不舒服了，虽然是现成的，但谁不想看着自己的名字不是，所以一气之下，决定删除用户，但是转念一想，他就是在这个用户下配置的环境，我删了，那配的环境不也就没有了，所以我就想着改一下用户名，在这期间有很多要改的地方，如果不小心操作错误，很有可能会无法登录系统，所以我在这里记录一下我的解决方法，免得以后踩坑。。。  
&emsp;&emsp;**二、** 我的主机操作系统被我换成kali了，当初想着root用户不安全，遂建立了一个普通用户，可是建立普通用户之后也发现，坑！太坑了！不输入sudo执行命令，很多命令都执行不了，而每次执行sudo命令时，都要输入当前用户密码，这就让我特别难受了。自认为密码长了就会安全，所以我给普通用户设置了一个15位长的密码，这每次执行命令的时候输一遍，谁顶得住，太麻烦了呀，于是想办法解决，功夫不负有心人，这个方法被我找到了，哈哈哈。。。
&emsp;&emsp;行，闲话扯皮完毕，下面进入正题。。。

## linux更改用户名
### （一）修改sudoer文件
&emsp;&emsp;我们为自己要改的名字提前赋予较高的权限，防止修改下面文件的过程中出现权限不足。（注：之前这个ubuntu用户名不是listone，我平时自己的代号是listone，所以我已经改过来了，当时修改的时候没有截图，那我就假设有一个aaa用户，我要将aaa用户名改为listone用户名）
![](http://wx1.sinaimg.cn/large/007cEDWily1g50kqajt7uj30bl01mjrb.jpg)
![](http://wx3.sinaimg.cn/large/007cEDWily1g50kpo69n7j30ak027glu.jpg)

### （二）修改shadow文件
&emsp;&emsp;这个文件中存储与登陆有关的内容格式如下：
```
username: passwd: lastchg: min: max: warn: inactive: expire: flag   
登录名:加密口令:最后一次修改时间:最小时间间隔:最大时间间隔:警告时间:不活动时间:失效时间:标志
```
![](http://wx2.sinaimg.cn/large/007cEDWily1g50kplsn5oj30au018dg1.jpg)
&emsp;&emsp;我们需要将登录名更改：
![](http://wx4.sinaimg.cn/large/007cEDWily1g50kpjko9vj30bs017wep.jpg)

### （三）开始修改目录
![](http://wx3.sinaimg.cn/large/007cEDWily1g50kph2bu6j30b101cjrb.jpg)
&emsp;&emsp;用户开始目录中，包含用户相关配置信息，我们要将二者相匹配

### （四）修改passwd文件
![](http://wx3.sinaimg.cn/large/007cEDWily1g50kpe77llj30bm01gdfs.jpg)
&emsp;&emsp;passwd文件内容格式如下：
```
用户名: 密码 : uid  : gid :用户描述：主目录：登陆shell
```
![](http://wx3.sinaimg.cn/large/007cEDWily1g50koxxp63j30c601maad.jpg)
&emsp;&emsp;我们将用户名，以及主目录等等改为新的名字
![](http://wx1.sinaimg.cn/large/007cEDWily1g50kpbx41rj30dx01bdg8.jpg)
### （五）如果还想修改原来用户下文件所属的组：
&emsp;&emsp;可以将 /etc/group 文件中的旧用户组，改为新的用户组
### （六）最后，再次进入/etc/sudoer 文件，将就用户名aaa 删除
![](http://wx1.sinaimg.cn/large/007cEDWily1g50kovmzzhj30b001r74g.jpg)
&emsp;&emsp;最后，重启之后就可以看到用户名已经更改。

## sudo用户小技巧
&emsp;&emsp;使用sudo用户时，可以修改sudoer文件，认真的同学，应该在上面已经发现了技巧，现在，在这里重提一下。
![](http://wx2.sinaimg.cn/large/007cEDWily1g50kopephcj309g01w74j.jpg)
&emsp;&emsp;就是在listone用户那里，加一个NOPASSWD就行了，比起修改用户名来说，很简单了。

## 总结
&emsp;&emsp;linux零碎的知识很多，在学习过程中，记录是很重要的，以前学到的东西，因为没有记录，当要用到的时候又给忘掉了，还要重新找资料，所以不如我们平时学习的时候多记笔记，多反思。
&emsp;&emsp;嗯，就是这样，这次分享就到这里了。