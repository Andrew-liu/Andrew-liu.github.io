title: 使用OwnCloud在VPS上搭建私有网盘
date: 2015-09-26 22:03:56
tags: Blog
---

> 终于找到了个时间, 把这个坑给填上..

# ownCloud介绍

> ownCloud is a self-hosted file sync and share server. It provides access to your data through a web interface, sync clients or WebDAV while providing a platform to view, sync and share across devices easily—all under your control. ownCloud's open architecture is extensible via a simple but powerful API for applications and plugins and it works with any storage.

`ownCloud`是一个`KDE`社区开发的免费软件，提供包括文件、音乐、日历、联系人的同步服务, 支持在服务器, PC, 手机上同步. 提供PC端, 手机端(Android/iOS)的同步工具.


> 有了自己的VPS, 那么多存储空间不能浪费呀, 开始折腾一个属于自己的私有云盘吧, 存放私密信息、小电影什么的, 你懂得.

<!--more-->


# 环境配置

使用平台:

- CentOS release 6.7 (Final)
- PHP 5.4.44

**第一次搭PHP的环境,还是有点折腾的,尤其注意在Centos中yum支持到PHP5.3版本安装**

**SSH连接自己的VPS**

```
$ ssh user@ip_address
```

**安装环境依赖**

```
# 下载所有依赖
$ yum install httpd mysql-server mysql-client php php-mysql php-curl


# 启动MySQL
$ service mysqld start
# 重启MySQL
$ service mysqld restart
# 设置root账户的密码
$ mysqladmin -u root password 'root'
# 密码登陆数据库
$ mysql -u root -p
# 创建新用户(最好不要用root账户操作)
$ grant all on cloud.* to hello@localhost identified by 'world';
# 创建一个ownCloud访问的数据库
mysql> create database cloud ;
```

**下载ownCloud**

```
$ cd ~
# 下载owncloud
$ wget https://download.owncloud.org/community/owncloud-8.1.0.tar.bz2

# 解压owncloud
$ tar -xf owncloud-8.1.0.tar.bz2

# 移动owncloud到Apache的工作目录
cp -r ~/owncloud /var/www/html

# 修改owncloud读写权限
$ chmod -R 777 /var/www/html/owncloud
```

**修改Apache服务器配置**

```
# 更改apache配置
$ vim /etc/httpd/conf/httpd.conf
```

找到如下位置:

```
# AllowOverride controls what directives may be placed in .htaccess files.
# It can be "All", "None", or any combination of the keywords:
#   Options FileInfo AuthConfig Limit
#
    AllowOverride None
```

修改为下面:

```
    AllowOverride All
```

**重新启动Apache服务**

```
# 重启Apache服务
$ service httpd restart
```

> 到此为此, VPS作为服务器的配置结束

# 配置ownCloud

使用其他PC的Web浏览器, 如Chrome访问`http:VPS_ip_address/owncloud`, 看到如下界面

![界面](http://ww2.sinaimg.cn/large/ab508d3dgw1euzne63yeaj20lq11eacz.jpg)

- 设置管理员账户
- 设置作为数据同步存储的目录
- 设置MySQL账户密码和对应的数据库


```
# 使用owncloud中data文件夹作为数据同步的位置
$ mkdir /var/www/html/owncloud/data
```

成功后可以看到ownCloud的Web同步文件管理

![成功](http://ww3.sinaimg.cn/large/ab508d3dgw1euznfw9p0tj21gu0nigo4.jpg)

> 同时可以下载PC客户端, Android手机客户端和iOS客户端进行文件同步.


# 报错及解决方案

错误: Web连接ip_address/owncloud出现PHP版本过低, 需要升级PHP版本到5.4


```
# 升级PHP版本到5.4+, 向yum中增加PHP5.4的仓库
$ rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
$ rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
# 使用新版本代替旧版本PHP
$ yum --enablerepo=remi,remi-test install httpd php php-common
$ yum --enablerepo=remi,remi-test install php-pecl-apc php-cli php-pear php-pdo php-mysql php-gd php-mbstring php-mcrypt php-xml
# 重启服务器
$ service httpd restart
```


错误: `xz compression not available`
原因是由于导入了错误的包版本链接, 清除添加的链接, 重新添加正确的链接

```
rpm -q epel-release
yum clean all
```






# 参考链接

- [How do I upgrade to the latest PHP version in CentOS with yum](http://serverfault.com/questions/456968/how-do-i-upgrade-to-the-latest-php-version-in-centos-with-yum)
- [Upgrading PHP on CentOS 6.5 (Final)](http://stackoverflow.com/questions/21502656/upgrading-php-on-centos-6-5-final)
