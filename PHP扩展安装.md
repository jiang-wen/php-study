>PHP扩展安装是PHP程序员的必备技能，在此通过本文向大家一起分享学习PHP扩展安装的经验，欢迎交流讨论

#### 一、查看已安装PHP扩展
##### 1. PHP命令方式查看

使用`php -m`命令查看：
`/usr/local/php/bin/php -m`
执行命令后显示结果如下：
![PHP Modules](https://upload-images.jianshu.io/upload_images/2305018-5ae2832cb8135c96.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
屏幕上列出来的就是已经安装的php扩展

##### 2.通过phpinfo()函数查看
在apache服务器web根目录新建一个php文件，输入如下代码：
```
<?php phpinfo(); ?>
```
Linux下可以使用如下命令：
`echo '<?php phpinfo(); ?>' > /usr/local/apache/htdocs/phpinfo.php`
上面这条命令使用管道符`echo`命令和管道符`>`新建了一个php文件


浏览器中打开`http://IP地址/phpinfo.php`，显示了PHP的信息，红框标记的就是已经安装的php扩展
![phpinfo信息](https://upload-images.jianshu.io/upload_images/2305018-5479855aaee70985.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 3.其他方式
查看php扩展安装情况的方式还有很多，如`get_loaded_extensions()`函数等..本文就不做一一介绍了。

#### 二、Linux安装PHP扩展

Linux安装php扩展的方式有pecl、yum、phpize等。本文主要介绍使用phpize方式安装；phpize安装和Linux软件源码安装方式很接近。

浏览器打开 http://pecl.php.net/，这里收录了绝大部分PHP扩展。下图第一个红框内可以搜索想要的扩展，第二个红框点击可以查看所有扩展包
![pecl](https://upload-images.jianshu.io/upload_images/2305018-4ecb19cf442cfcde.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

搜索redis得到一个结果如下：
![搜索redis](https://upload-images.jianshu.io/upload_images/2305018-161cba661b60ef93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击进入后可以看到各版本的redis，在Downloads列里有tgz格式的包和dll格式的包，分别为linux和windows下需要安装的格式
![redis下载列表](https://upload-images.jianshu.io/upload_images/2305018-ae9a2d9ac6d05799.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

redis安装的时候需要注意选择对应php的版本的包，此网页最下方有版本要求的信息

![redis环境依赖](https://upload-images.jianshu.io/upload_images/2305018-5995cff1372e11e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

作者安装的是php7.2.9，选择最新版[redis-4.2.0.tgz](http://pecl.php.net/get/redis-4.2.0.tgz) 右键单击复制链接

依次执行以下Linux命令
当前目录：`cd /root`
下载：`wget http://pecl.php.net/get/redis-4.2.0.tgz`
解压缩：`tar -zxvf redis-4.2.0.tgz`
进入目录：`cd redis-4.2.0`
执行`phpize`命令：`/usr/local/php/bin/phpize`

附带php-config的路径执行./configure：
`./configure --with-php-config=/usr/local/php/bin/php-config`

编译安装：`make && make install`

安装完成后在php.ini配置文件中添加一行
`extension=redis.so`
最后重启apache，php_redis扩展就安装完成了

可以使用前文介绍的方式查看是否安装成功
![安装成功](https://upload-images.jianshu.io/upload_images/2305018-90fd971e5c64286c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 三、Windows安装PHP扩展

有了Linux下安装PHP扩展的经验和基础，在Windows下安装PHP扩展就比较方便了。

从 http://pecl.php.net/ 下载符合php版本的dll文件放入PHP安装目录下的`ext`文件夹内，修改php.ini配置，添加一行`extension=php_redis.dll`（此处要注意文件名一致），重启Apache就完成了