# 前提

## 添加阿里yum源

```
打开 mirrors.aliyun.com，选择centos的系统，可按照这个网站的指示进行操作，下面是我将网站内容复制过来的,也可按下面步骤进行操作
```

### 配置方法

#### 1、备份

```shell
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
```

#### 2、下载新的 CentOS-Base.repo 到 /etc/yum.repos.d/

```shell
CentOS 6
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo
或者
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-6.repo

CentOS 7
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
或者
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo

CentOS 8
wget -O /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
或者
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
```

#### 3、运行 yum makecache 生成缓存

```shell
yum makecache
```

#### 4、安装wget（可选）

```shell
yum install wget
```

# 安装mysql5.7

## 1、下载并安装mysql官方的yum repository

```shell
1、下载
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm

2、安装
yum -y install mysql57-community-release-el7-10.noarch.rpm
```

## 2、安装mysql 服务器

```shell
yum -y install mysql-community-server
```

## 3、启动mysql服务

```shell
service mysqld start
```

## 4、获取mysql root用户临时密码

```shell
grep "password" /var/log/mysqld.log
```

## 5、修改root密码

```shell
1、使用上一步获取的临时密码登录mysql
mysql -uroot -p

2、修改密码
alter user 'root'@'localhost' identified by 'root';
```

## 6、mysql相关设置

### 开启mysql的远程访问

```shell
(应该就是对mysql的mysql这个数据库下的user表中的host字段做出相应的修改，所以如果是8.0的话应该可以直接使用update进行修改，就是更新这个user表嘛)
1、grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;

2、刷新权限
flush privileges
```

### 为firewalld添加开放端口

```shell
1、3306端口
firewall-cmd --zone=public --add-port=3306/tcp --permanent
2、8080端口
firewall-cmd --zone=public --add-port=8080/tcp --permanent
3、重新载入
firewall-cmd --reload
```

### 修改mysql字符集为utf8

```
(默认的mysql的my.ini配置文件所在的位置：/etc/my.ini)
新加下面三行

#在[mysqld]部分添加：
character-set-server=utf8
#在文件末尾新增[client]段，并在[client]段添加：
[client]
default-character-set=utf8
```

