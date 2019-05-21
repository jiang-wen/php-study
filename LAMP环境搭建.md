LAMP环境搭建是PHP程序员必须掌握的一项基本技能，但是对于初学者来说，操作步骤比较繁琐。本文是作者个人对LAMP环境搭建的整个流程的理解，在此学习分享，希望对学习PHP的同学有所帮助，欢迎指正错误哦~

#### 一、 Linux安装（CentOS）

如果是本地测试环境，则需要安装虚拟机软件，这里使用的是[VirtualBox](https://www.virtualbox.org/)，下载好后根据自己需要安装在相应的位置就好了。

[CentOS](https://www.centos.org/)的`ISO`文件下载好以后用虚拟机打开安装。

相关文章：[Windows下利用VirtualBox安装CentOS虚拟机](https://www.jianshu.com/p/150a48ec6436)

#### 二、安装环境准备

1. 更新yum源： `yum -y update`
2. 安装wget 和 vim： `yum -y install wget vim`
wget用于下载文件，本教程下载的都是`tar.gz`格式的压缩包；
vim为编辑器
3. 安装编译环境：
 `yum -y install gcc gcc-c++ libxml2-devel libtool expat-devel automake autoconf make`
这里安装了c和c++的编译环境以及可能需要用到的库
4. **(重要)本教程使用root用户执行所有指令，如果使用其他用户，某些步骤可能需要添加`sudo`命令才能执行；所有文件下载均在`/root`目录下，请注意每一步骤执行时所在的目录。**


#### 三、Apache 安装 


1. **apr 和 apr-util 下载安装**
[apr官网](https://apr.apache.org/)可以下载到apr和apr-util的源码
本教程下载的是[apr-1.6.5.tar.gz](http://mirrors.hust.edu.cn/apache/apr/apr-1.6.5.tar.gz)和[apr-util-1.6.1.tar.gz](http://mirrors.hust.edu.cn/apache/apr/apr-util-1.6.1.tar.gz)
进入root目录：`cd /root`
下载使用wget：
`wget http://mirrors.hust.edu.cn/apache/apr/apr-1.6.5.tar.gz`
`wget http://mirrors.hust.edu.cn/apache/apr/apr-util-1.6.1.tar.gz`

      下载完成后开始解压并编译安装apr-1.6.5
    ```
    tar -zvxf apr-1.6.5.tar.gz
    cd apr-1.6.5
    ./configure --prefix=/usr/local/apr
    make && make install
    ```
    再安装apr-util-1.6.1
    ```
    cd /root
    tar -zvxf apr-util-1.6.1.tar.gz
    cd apr-util-1.6.1
    ./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
    make && make install
    ```

>除了编译安装方式，也可以在编译httpd时，将apr 和apr-util解压到httpd源码目录下的`srclib`目录，分别命名为`apr`和`apr-util`；
>执行httpd`./configure`时，将 `--with-apr=/usr/local/apr`  和 `--with-apr-util=/usr/local/apr-util`两个配置项改为   `--with-apr-included`执行

2. **pcre 下载安装**
进入root目录：`cd /root`
pcre可以从[Index of pcre](https://ftp.pcre.org/pub/pcre/)下载
`wget https://ftp.pcre.org/pub/pcre/pcre-8.42.tar.gz`
下载完成后解压编译安装
    ```
    tar -zvxf pcre-8.42.tar.gz
    cd pcre-8.42
    ./configure
    make && make install
    ```
3. **zlib 下载安装**
进入root目录：`cd /root`
从[zlib官网](http://www.zlib.net/)找到下载链接
`wget http://www.zlib.net/zlib-1.2.11.tar.gz`
下载完成后解压安装
    ```
    tar -zxvf zlib-1.2.11.tar.gz
    cd zlib-1.2.11
    ./configure
    make && make install
    ```
4. **openssl 下载安装**
进入root目录：`cd /root`
[Index of openssl](http://distfiles.macports.org/openssl/)上可以下载openssl源码包
`wget http://distfiles.macports.org/openssl/openssl-1.0.2q.tar.gz`
下载完成后解压安装
    ```
    tar -zxvf openssl-1.0.2q.tar.gz
    cd openssl-1.0.2q
    ./config shared zlib
    make && make install
    ```

5. **httpd 下载安装**
进入root目录：`cd /root`
[Apache官网](http://httpd.apache.org/)上可以下载到最新版本的源码
下载：`wget http://mirror.bit.edu.cn/apache/httpd/httpd-2.4.38.tar.gz`
解压：`tar -zvxf httpd-2.4.38.tar.gz`
解压完成进入httpd-2.4.38目录：`cd httpd-2.4.38`
配置编译安装选项：
`./configure --prefix=/usr/local/apache --with-zlib=/usr/local/zlib --with-apr=/usr/local/apr --with-apr-util=/usr/local/apr-util --with-ssl=/usr/local/ssl --enable-so --enable-dav --enable-ssl --enable-rewrite --enable-modules=most --enable-maintainer-mode`
编译安装：`make && make install`

至此apache的安装就完成了
执行命令启动apache `/usr/local/apache/bin/apachectl start`
启动的时候可能会报错:
```
httpd: Syntax error on line 134 of /usr/local/apache/conf/httpd.conf: Cannot load modules/mod_ssl.so into server: libssl.so.1.0.0: cannot open shared object file: No such file or directory
```
解决方法,在`/etc/ld.so.conf`文件中写入openssl库文件的搜索路径:
`echo "/usr/local/lib64" >> /etc/ld.so.conf`
再使用`ldconfig -v`命令查看动态链接生效结果

>如果动态链接之后依然不生效，检查一下`/usr/local/lib64`文件夹是否存在，某些系统可能是`/usr/local/lib`；检查的时候顺便进lib目录里查看`libssl.so.1.0.0`和`libcrypto.so.1.0.0 `是否存在，如果不存在的话需要到openssl源码目录下复制过来；
```
cd  /root/openssl-1.0.2q
cp libssl.so.1.0.0 /usr/local/lib64
cp libcrypto.so.1.0.0  /usr/local/lib64
ldconfig -v
```

再次执行命令启动Apache，如果再出现以下错误，是httpd.conf配置的原因，暂时先不管
```
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using localhost.localdomain. Set the 'ServerName' directive globally to suppress this message
```

在浏览器输入服务器IP地址访问，显示如下则安装成功
![安装成功](https://upload-images.jianshu.io/upload_images/2305018-429673ebb57e2f7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

若服务器无响应，有可能是防火墙80端口未开放；对于云服务器也有可能是因为没有配置[安全组规则](https://support.huaweicloud.com/usermanual-vpc/SecurityGroup_0012.html)。
开放80端口：`firewall-cmd --zone=public --add-port=80/tcp --permanent`
重启防火墙：`systemctl restart firewalld.service`


#### 三、PHP安装
下载php源码，可以从php官方 [https://secure.php.net/releases/](https://secure.php.net/releases/)找到各版本下载链接

进入root目录：`cd /root`
开始下载：`wget https://www.php.net/distributions/php-7.2.9.tar.gz`

解压：`tar -zxvf php-7.2.9.tar.gz`
进入目录：`cd php-7.2.9`
配置安装目录和模块加载等信息：
`./configure --prefix=/usr/local/php --with-apxs2=/usr/local/apache/bin/apxs --with-config-file-path=/usr/local/php --with-mysql=mysqlnd --with-mysqli=mysqlnd --with-pdo-mysql=mysqlnd --enable-mbstring`

php7已经不再支持使用mysql扩展，因此配置时会出现如下提示，可忽略：
```
configure: WARNING: unrecognized options: --with-mysql
```
接下来编译安装：`make && make install`

编译完成后使用vim新建一个测试文件：`vim phpinfo.php`
按`a`键进入编辑模式，输入测试代码：
```
<?php echo phpinfo();?>
```
再按下`ESC`回到命令模式，输入`:wq`命令后按`enter`键保存并退出
输入`/usr/local/php/bin/php phpinfo.php`命令执行测试文件
返回phpinfo结果如下则php安装成功
![phpinfo](https://upload-images.jianshu.io/upload_images/2305018-36bf1263439d2351.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 四、MySQL安装

##### **1.检测是否安装MySQL**

Linux平台上推荐使用RPM包来安装MySQL
安装前，我们可以检测系统是否自带安装 MySQL:
`rpm -qa | grep mysql`
如果你系统有安装，那可以选择进行卸载:
普通删除模式 ：`rpm -e mysql`　
如果使用普通删除命令删除时，提示有依赖的其它文件，则用强力删除模式：`rpm -e --nodeps mysql` 可以对其进行强力删除　

##### **2.安装MySQL**

使用wget下载rpm包 `wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm`
再依次执行以下命令用yum进行安装
```
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum -y update
yum -y install mysql-server
```

安装完成后进行权限设置：
`chown mysql:mysql -R /var/lib/mysql`

启动MySQL:
`systemctl start mysqld`
查看MySQL状态：
`systemctl status mysqld`

如果我们是第一次启动 mysql 服务，mysql 服务器首先会进行初始化的配置。

##### **3.验证MySQL安装**

使用 mysqladmin 工具来获取服务器状态：`mysqladmin --version`
显示结果基于你的系统信息：
```
mysqladmin  Ver 8.42 Distrib 5.6.43, for Linux on x86_64
```
使用MySQL客户端命令连接至MySQL服务器：`mysql`
出现如下界面则连接成功：
```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.6.43 MySQL Community Server (GPL)

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```


使用`exit;`命令断开连接

##### **4.修改密码**

MySQL安装成功后，默认root密码为空，需要进行设置
使用命令修改root密码：`mysqladmin -u root password "12345678";`
可能会提示如下安全风险，可忽略：
```
Warning: Using a password on the command line interface can be insecure.
```

输入命令登录MySQL `mysql -uroot -p`，使用该命令后会提示输入密码，输入即可

登陆成功后可以使用MySQL命令对MySQL数据库进行操作了。

##### **5.远程管理MySQL**

如果需要**远程管理数据库**，则需要用户有远程访问权限，即`user`表`host`字段的值为`%`

可以使用下面的命令新建一个具有全部权限的用户 remote_admin：

`GRANT ALL PRIVILEGES ON *.* TO 'remote_admin' IDENTIFIED BY '12345678' WITH GRANT OPTION;`

如果还不能远程连接，有可能是防火墙3306端口未开放

开放3306端口：`firewall-cmd --zone=public --add-port=3306/tcp --permanent`
重启防火墙：`systemctl restart firewalld.service`


#### 五、配置Apache和PHP

使用vim打开Apache的配置文件：
`vim /usr/local/apache/conf/httpd.conf`

##### 1.添加PHP支持：

在文件中添加一行；如果PHP版本为5.X，则需要加载PHP5的模块
```
LoadModule php7_module        modules/libphp7.so
```


在`DirectoryIndex` 后添加 `index.php`
```
 DirectoryIndex index.html index.php
```
在`<IfModule mime_module>`下添加两行
```
AddType application/x-httpd-php .php .phtml
AddType application/x-httpd-php-source .phps
```


##### 2.修改ServerName

去掉`ServerName`前的`#`号，将其改成如下形式：
```
ServerName localhost
```
##### 3.开启Apache支持伪静态

将rewrite模块加载前的`#`号去掉
```
LoadModule rewrite_module modules/mod_rewrite.so
```
将`<Directory "/usr/local/apache/htdocs">`标签下的 `AllowOverride None`修改为
```
AllowOverride all
```
##### 4.不显示目录结构

将`<Directory "/usr/local/apache/htdocs">`标签下的 `Options Indexes FollowSymLinks`修改为
```
Options  FollowSymLinks
```
##### 5.修改php.ini


我们在安装PHP的时候设置了配置文件的路径
`--with-config-file-path=/usr/local/php`
但是目前`/usr/local/php/`路径下并没有`php.ini`文件，而在解压目录`/root/php-7.2.9`下有两个php配置文件
分别是`php.ini-development`和`php.ini-production`
这两个文件是开发环境和生产环境的默认配置，这里我们使用开发环境
使用`cp`命令复制：
```
cp /root/php-7.2.9/php.ini-development /usr/local/php/php.ini
```

##### 6.验证服务器能否解析PHP
首先重启Apache：
`/usr/local/apache/bin/apachectl restart`
没有返回任何结果，则表明重启成功

使用管道符`>`新建一个测试文件
`echo '<?php phpinfo();' > /usr/local/apache/htdocs/phpinfo.php`

打开浏览器输入地址`http://IP地址/phpinfo.php`
显示phpinfo信息则配置成功


