# Linux

```
未整理内容：
修改服务器时间：`date -s "2019/10/31 23:57:00"`
改回当前时间[NTP服务器(上海)]：`ntpdate -u ntp.api.bz`
```

## 查看磁盘使用情况

- df
  - 以字节展示：df -l
  - 可读性展示：df -h
- du
  - 可读性展示（单位1024）：du -h
  - 可读性展示（单位1000）：du -H
  - 查看当前目录大小：du -hd0 /home/
  - 查看当前目录及子目录大小：du -hd1 /home/boot/

## 查看系统信息uname

- 全部信息: uname -a
  
  - `Linux dudu 3.10.0-514.26.2.el7.x86_64 #1 SMP Tue Jul 4 15:04:05 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux`

- 硬件平台: uname -i
  
  - `x86_64`

- 机器硬件（CPU）名: uname -m
  
  - `x86_64`

- 节点名称: uname -n
  
  - `dudu`

- 操作系统: uname -o
  
  - `GNU/Linux`

- 系统处理器的体系结构: uname -p
  
  - `x86_64`

- 操作系统的发行版号 uname -r
  
  - `3.10.0-514.26.2.el7.x86_64`

- 系统名: uname -s
  
  - `Linux`

- 内核版本: uname -v
  
  - `#1 SMP Tue Jul 4 15:04:05 UTC 2017`

- 内核版本详细信息（更直观）: lsb_release -a
  
  > LSB(Linux Standard Base) 此命令适用于所有的Linux发行版本
  
  ```
  LSB Version:    :core-4.1-amd64:core-4.1-noarch
  Distributor ID: CentOS
  Description:    CentOS Linux release 7.7.1908 (Core)
  Release:        7.7.1908
  Codename:       Core
  ```

## 命令的使用lsof

1.列出所有打开的文件:

- lsof
- 备注: 如果不加任何参数，就会打开所有被打开的文件，输出内容非常多

2.查看谁正在使用某个文件

- lsof /filepath/file

3.递归查看某个目录的文件信息

- lsof +D /filepath/filepath2/
- 备注: 使用了+D，对应目录下的所有子目录和文件都会被列出

4.不使用+D选项，遍历查看某个目录的所有文件信息 的方法

- lsof | grep '/filepath/filepath2/'

5.列出某个用户打开的文件信息

- lsof -u username
- 备注: -u 选项，u其实是user的缩写
- 查看所有用户 vim /etc/group

6.列出某个程序所打开的文件信息

- lsof -c mysql
- 备注: -c 选项将会列出所有以mysql开头的程序的文件，其实你也可以写成lsof | grep mysql,但是第一种方法明显比第二种方法要少打几个字符了

7.列出多个程序多打开的文件信息

- lsof -c mysql -c apache

8.列出某个用户以及某个程序所打开的文件信息

- lsof -u test -c mysql

9.列出除了某个用户外的被打开的文件信息

- lsof -u ^root
- 备注：^这个符号在用户名之前，将会把是root用户打开的进程不让显示

10.通过某个进程号显示该进行打开的文件

- lsof -p 1

11.列出多个进程号对应的文件信息

- lsof -p 123,456,789

12.列出除了某个进程号，其他进程号所打开的文件信息

- lsof -p ^1

13.列出所有的网络连接

- lsof -i

14.列出所有tcp 网络连接信息

- lsof -i tcp

15.列出所有udp网络连接信息

- lsof -i udp

16.***列出谁在使用某个端口（重点）***

- lsof -i:3306

17.列出谁在使用某个特定类型的端口

- 特定的udp端口
  - lsof -i udp:55
- 特定的tcp端口
  - lsof -i tcp:80

18.列出某个用户的所有活跃的网络端口

- lsof -a -u test -i

19.列出所有网络文件系统

- lsof -N

20.域名socket文件

- lsof -u

21.某个用户组所打开的文件信息

- lsof -g 5555

22.根据文件描述列出对应的文件信息

- lsof -d description(like 2)

23.根据文件描述范围列出文件信息

- lsof -d 2-3

## 定时任务cron

```
# test cron 每分钟执行一次
* * * * * echo "do rm -rf lu*.tmp $(date +\%Y-\%m-\%d~\%H:\%M:\%S)" >> /tmp/rm-tmp-job.txt
# this is rm liberoffice generate lu*.tmp job
0 4 */5 * * echo "$(date +\%Y-\%m-\%d~\%H:\%M:\%S) do rm -rf /tmp/lu*.tmp" >> /tmp/rm-tmp-job.txt
0 4 */5 * * rm -rf /tmp/lu*.tmp
```

## Secure(CRT/FX)使用技巧

> Linux shell ftp 工具，推荐使用 SecureCRT SecureFX

- 批量导入服务器
  
  - 先决文件：https://www.vandyke.com/support/tips/importsessions.html
  
  - 创建一个.csv文件,写上对应服务器信息,如下
    
    ```
    protocol,username,folder,session_name,hostname
    SSH2,root,hello,pro-hello,127.0.0.1
    ```

## Linux目录释义

### 6.1 前言

- /usr：系统级的目录，可以理解为C:/Windows/；/usr/lib理解为C:/Windows/System32。
- /usr/local：用户级的程序目录，可以理解为C:/Progrem Files/。用户自己编译的软件默认会安装到这个目录下。
- /opt：用户级的程序目录，可以理解为D:/Software，opt有可选的意思，这里可以用于放置第三方大型软件（或游戏），当你不需要时，直接rm -rf掉即可。在硬盘容量不够时，也可将/opt单独挂载到其他磁盘上使用。

### 6.2 需要注意的

> 在 Linux 系统中，有几个目录是比较重要的，平时需要注意不要误删除或者随意更改内部文件。

- /etc：上边也提到了，这个是系统中的配置文件，如果你更改了该目录下的某个文件可能会导致系统不能启动。
- /bin, /sbin, /usr/bin, /usr/sbin: 这是系统预设的执行文件的放置目录，比如 ls 就是在/bin/ls 目录下的。
  - /bin, /usr/bin 是给非root用户使用的指令；
  - /sbin, /usr/sbin 则是给root使用的指令。
- /var：这是一个非常重要的目录，系统上跑了很多程序，那么每个程序都会有相应的日志产生，而这些日志就被记录到这个目录下，具体在/var/log 目录下，另外mail的预设放置也是在这里。

### 6.3 详细目录

- /usr：
  - 可不是user目录呦。实为unix system resources（unix系统资源）的缩写。
- /bin：
  - bin是Binary的缩写, 这个目录存放着最经常使用的命令。
- /boot：
  - 这里存放的是启动Linux时使用的一些核心文件，包括一些连接文件以及镜像文件。
- /dev ：
  - dev是Device(设备)的缩写, 该目录下存放的是Linux的外部设备，在Linux中访问设备的方式和访问文件的方式是相同的。
- /etc：
  - 这个目录用来存放所有的系统管理所需要的配置文件和子目录。
- /home：
  - 用户的主目录，在Linux中，每个用户都有一个自己的目录，一般该目录名是以用户的账号命名的。
- /lib：
  - 这个目录里存放着系统最基本的动态连接共享库，其作用类似于Windows里的DLL文件。几乎所有的应用程序都需要用到这些共享库。
- /lost+found：
  - 这个目录一般情况下是空的，当系统非法关机后，这里就存放了一些文件。
- /media：
  - linux系统会自动识别一些设备，例如U盘、光驱等等，当识别后，linux会把识别的设备挂载到这个目录下。
- /mnt：
  - 系统提供该目录是为了让用户临时挂载别的文件系统的，我们可以将光驱挂载在/mnt/上，然后进入该目录就可以查看光驱里的内容了。
- /opt：
  - 这是给主机额外安装软件所摆放的目录。比如你安装一个ORACLE数据库则就可以放到这个目录下。默认是空的。
- /proc：
  - 这个目录是一个虚拟的目录，它是系统内存的映射，我们可以通过直接访问这个目录来获取系统信息。
  - 这个目录的内容不在硬盘上而是在内存里，我们也可以直接修改里面的某些文件，比如可以通过下面的命令来屏蔽主机的ping命令，使别人无法ping你的机器：
  - echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all
- /root：
  - 该目录为系统管理员，也称作超级权限者的用户主目录。
- /sbin：
  - s就是Super User的意思，这里存放的是系统管理员使用的系统管理程序。
- /selinux：
  - 这个目录是Redhat/CentOS所特有的目录，Selinux是一个安全机制，类似于windows的防火墙，但是这套机制比较复杂，这个目录就是存放selinux相关的文件的。
- /srv：
  - 该目录存放一些服务启动之后需要提取的数据。
- /sys：
  - 这是linux2.6内核的一个很大的变化。该目录下安装了2.6内核中新出现的一个文件系统 sysfs 。
  - sysfs文件系统集成了下面3种文件系统的信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。
  - 该文件系统是内核设备树的一个直观反映。
  - 当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中被创建。
- /tmp：
  - 这个目录是用来存放一些临时文件的。
- /usr/bin：
  - 系统用户使用的应用程序。
- /usr/sbin：
  - 超级用户使用的比较高级的管理程序和系统守护程序。
- /usr/src：
  - 内核源代码默认的放置目录。
- /var：
  - 这个目录中存放着在不断扩充着的东西，我们习惯将那些经常被修改的目录放在这个目录下。包括各种日志文件。
- /run：
  - 是一个临时文件系统，存储系统启动以来的信息。当系统重启时，这个目录下的文件应该被删掉或清除。如果你的系统上有 /var/run 目录，应该让它指向 run。

## linux服务器重启指令

### Linux 的五个重启命令

　　1、shutdown

　　2、poweroff

　　3、init

　　4、reboot

　　5、halt

### 五个重启命令的具体说明

　　shutdown

　　reboot

　　在linux下一些常用的关机/重启命令有shutdown、halt、reboot、及init，它们都可以达到重启系统的目的，但每个命令的内部工作过程是不同的，通过本文的介绍，希望你可以更加灵活的运用各种关机命令。

#### **1.shutdown**

　　shutdown命令安全地将系统关机。 有些用户会使用直接断掉电源的方式来关闭linux，这是十分危险的。因为linux与windows不同，其后台运行着许多进程，所以强制关机可能会导致进程的数据丢失﹐使系统处于不稳定的状态﹐甚至在有的系统中会损坏硬件设备。而在系统关机前使用shutdown命令﹐系统管理员会通知所有登录的用户系统将要关闭。并且login指令会被冻结﹐即新的用户不能再登录。直接关机或者延迟一定的时间才关机都是可能的﹐还可能重启。这是由所有进程〔process〕都会收到系统所送达的信号〔signal〕

　　决定的。这让像vi之类的程序有时间储存目前正在编辑的文档﹐而像处理邮件〔mail〕和新闻〔news〕的程序则可以正常地离开等等。

　　shutdown执行它的工作是送信号〔signal〕给init程序﹐要求它改变runlevel。

　　Runlevel 0被用来停机〔halt〕﹐runlevel 6是用来重新激活〔reboot〕系统﹐而runlevel 1则是被用来让系统进入管理工作可以进行的状态﹔这是预设的﹐假定没有-h也没有-r参数给shutdown。要想了解在停机〔halt〕或者重新开机〔reboot〕过程中做了哪些动作﹐你可以在这个文件/etc/inittab里看到这些runlevels相关的资料。

　　**shutdown** 参数说明:

　　[-t] 在改变到其它runlevel之前﹐告诉init多久以后关机。

　　[-r] 重启计算器。

　　[-k] 并不真正关机﹐只是送警告信号给

　　每位登录者〔login〕。

　　[-h] 关机后关闭电源〔halt〕。

　　[-n] 不用init﹐而是自己来关机。不鼓励使用这个选项﹐而且该选项所产生的后果往往不总是你所预期得到的。

　　[-c] cancel current process取消目前正在执行的关机程序。所以这个选项当然没有时间参数﹐但是可以输入一个用来解释的讯息﹐而这信息将会送到每位使用者。

　　[-f] 在重启计算器〔reboot〕时忽略fsck。

　　[-F] 在重启计算器〔reboot〕时强迫fsck。

　　[-time] 设定关机〔shutdown〕前的时间。

#### **2.halt**----最简单的关机命令

　　其实halt就是调用shutdown -h。halt执行时﹐杀死应用进程﹐执行sync系统调用﹐文件系统写操作完成后就会停止内核。

　　参数说明:

　　[-n] 防止sync系统调用﹐它用在用fsck修补根分区之后﹐以阻止内核用老版本的超级块〔superblock〕覆盖修补过的超级块。

　　[-w] 并不是真正的重启或关机﹐只是写

　　wtmp〔/var/log/wtmp〕纪录。

　　[-d] 不写wtmp纪录〔已包含在选项[-n]中〕。

　　[-f] 没有调用shutdown而强制关机或重启。

　　[-i] 关机〔或重启〕前﹐关掉所有的网络接口。

　　[-p] 该选项为缺省选项。就是关机时调用poweroff。

#### **3.reboot**

　　reboot的工作过程差不多跟halt一样﹐不过它是引发主机重启﹐而halt是关机。它 的参数与halt相差不多。

#### **4.init**

　　init是所有进程的祖先﹐它的进程号始终为1﹐所以发送TERM信号给init会终止所有的 用户进程﹑守护进程等。shutdown 就是使用这种机制。init定义了8个运行级别(runlevel)， init 0为关机﹐init 1为重启。关于init可以长篇大论﹐这里就不再叙述。另外还有telinit命令可以改变init的运行级别﹐比如﹐telinit -iS可使系统进入单用户模式﹐ 并且得不到使用shutdown时的信息和等待时间。