# 虚拟环境

```
指一个假的、使用软件模拟出来的、可供软件运行的环境

VMWare workstation:可以为我们提供一个虚拟环境
我们在这个虚拟环境中，可以安装多台虚拟机（虚拟机相当于裸机）
我们可以在虚拟机中安装操作系统
```

# 创建虚拟机

1、文件->创建虚拟机

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322085700.png)

2、选择“自定义”

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322085810.png)

3、使用默认配置，直接下一步

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322085950.png)

4、选择“稍后安装操作系统”

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322090057.png)

5、安装“CentOS 7 64位”

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322090212.png)

6、修改虚拟机名称、以及位置

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322090318.png)

7、使用默认配置，直接下一步

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322090318.png)

8、为虚拟机分配内存，使用默认1G的大小就可以了(不要小于726M，不然在安装操作系统的时候，使用的是命令行来安装的，而不是图形化界面)

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322090440.png)

9、虚拟机的网络配置，选择NAT模式

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322090531.png)

10、使用默认值，直接下一步

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322090930.png)

11、使用默认值，直接下一步

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322091038.png)

12、选择“创建新虚拟磁盘”

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322091117.png)

13、(最大磁盘大小设置为100G，避免后期使用的时候，磁盘空间不够，需要重新挂载磁盘，麻烦)

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322091253.png)

14、使用默认配置，直接下一步

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322091412.png)

# 安装CentOS 7 64位操作系统

1、点击红框中的文字，打开虚拟机设置

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322092043.png)

2、(安装minimal版本的就可以了，这个版本需要使用命令行操作linux系统，而没有图形化界面)

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322092230.png)

3、

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322092346.png)

4、

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322092514.png)

5、修改时区,选择上海

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322092916.png)

![]()![QQ截图20210322093003](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322093003.png)

6、修改hostname

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322093057.png)

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322093251.png)

7、创建磁盘分区

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322093420.png)

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322095714.png)

然后再点击done，进入下面的界面

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322093623.png)

## 磁盘分区

```
创建三个分区
1、boot（/boot）-》存放linux内核启动时需要的引导程序-》200M
2、swap-》交换分区，当一个程序需要运行，但是内存不足时，此时OS可以将内存中的部分数据放在swap区，从而腾出部分内存区域-》设置为虚拟机内存大小的两倍（由于前面我们为其分配了1G内存，所以，这里设置为2G）
3、用户区（/）-》放在用户数据、程序等
```

### /boot

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322094735.png)

### swap

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322094840.png)

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322094958.png)

### /

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322095042.png)

容量那一栏不用填写，表示分配所有的剩余容量

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322095500.png)

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322095610.png)

8、设置root用户密码（123456）

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322095934.png)

# 虚拟环境的网络配置

1、cd /etc/sysconfig/network-scripts/  

打开ifcfg-ens33文件

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\图片1.png)

修改下面配置

BOOTPROTO=static (表示使用静态配置的ip)

ONBOOT=yes

添加下面配置项 (打开vmware的虚拟网络编辑器，找到子网ip)

IPADDR=[子网ip的前三段+主机号]（注意：主机号不能是0和255，也不能是1——被vmware虚拟出来的网卡所占用，也不能是2，被网关所占用）

GATEWAY=[前三段与子网ip相同].2

NETMASK=255.255.255.0

DNS1=114.114.114.114

最后，删除UUID

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322113839.png)

2、service network restart

3、如果ping [www.baidu.com可行](http://www.baidu.com可以)就没问题

# 修改主机名

```
修改主机名
hostname 查看主机名
hostnamectl set-hostname xxx 修改主机名
然后重连
```

# yum仓库配置

```
https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.3e221b11BJaPbf
```

# 安装JDK1.8

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\图片3.png)

```
如果出现上述错误
执行：yum install glibc.i686
```

# 克隆虚拟机

1、==首先需要关闭虚拟机！！！==

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322113211.png)

2、克隆

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322113342.png)

3、创建链接克隆（选用这种方式，快）

![](F:\zehua\personalFiles\newStart\其他\linux学习\linux虚拟机图片\QQ截图20210322113405.png)

4、开启克隆的虚拟机，进行相关配置

```
1、修改ip地址
cd /etc/sysconfig/network-scripts/  
打开ifcfg-ens33文件

2、删除UUID、网卡mac地址（如果有的话）

3、修改hostname
hostname 查看主机名
hostnamectl set-hostname xxx 修改主机名

4、删除/etc/udev/rule.d/70-persistent-net.rules文件（如果有的话）
这个文件中保存了网卡物理地址与网卡名的关系

5、reboot重启
```

