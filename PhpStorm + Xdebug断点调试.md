一、安装Xdebug
---
在[Xdebug官网下载页](https://xdebug.org/download.php)可以下载xdebug，可以根据自己的PHP版本下载对应的xdebug版本，需要注意的有三个关键点：
* PHP版本号 5.x 还是 7.x
* 32位还是64位
* TS（线程安全）还是NTS（非线程安全）

如果不太清楚自己的PHP版本,可以点击下图所示红框内的链接，分析phpinfo打印的信息，自动提供对应的下载版本


![xdebug下载](https://upload-images.jianshu.io/upload_images/2305018-7fe09129aa1165c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


浏览器打开phpinfo信息，`ctrl + a`全选 ，`ctrl + c` 复制

![phpinfo](https://upload-images.jianshu.io/upload_images/2305018-b5d4cb256e290b85.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`ctrl + v`粘贴到分析页面的输入框内，点击分析按钮

![分析phpinfo](https://upload-images.jianshu.io/upload_images/2305018-8e121b8ac02ddf36.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

按照分析结果给出的指示，一步一步完成就好了。这里分析后的路径都是正确的，对应了你本机的路径。
![分析结果](https://upload-images.jianshu.io/upload_images/2305018-e067f188f8945728.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![php.ini配置](https://upload-images.jianshu.io/upload_images/2305018-3310a42ab3c6ce0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

重启服务器后再次打开phpinfo，验证安装是否成功，`ctrl + F` 搜索 xdebug关键字，显示如下版本信息即安装成功

![xdebug](https://upload-images.jianshu.io/upload_images/2305018-af4b1c2033e0fa89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

二、配置xdebug
---
本文开启的配置如下
```
[XDebug]
xdebug.profiler_output_dir="D:\phpStudy\tmp\xdebug"
xdebug.trace_output_dir="D:\phpStudy\tmp\xdebug"
zend_extension="D:\phpStudy\php\php-7.1.27\ext\php_xdebug.dll"

xdebug.remote_enable = 1
xdebug.remote_host = localhost
xdebug.remote_mode = req
xdebug.remote_handler = dbgp
xdebug.remote_port = 9000
xdebug.idekey = PHPSTORM


```

####【xdebug配置说明】


**开启Xdebug方式的错误提示：**
只要启用了xdebug扩展那么该功能是默认开启的，将以xdebug的风格进行错误显示，如果想以原php的风格显示可以在配置文件中设置：xdebug.default_enable=0，注意该设置项并不是指关闭xdebug的所有功能

**配置大变量的显示：**
在有些程序中变量会很大，比如著名内容管理系统drupal8中的节点渲染数组，如果直接`print_r`那么可以将内存8G的I5计算机卡死，所以Xdebug提供了配置大变量打印配置以避免这种情况，配置以下配置项：
`xdebug.var_display_max_children`
整数，默认`128`，显示的数组子元素或对象属性的最大数量，不限制则设定为`-1`，远程调试时不受影响
`xdebug.var_display_max_data`
整数，默认`512`，显示字符串的最大长度，不限设置为`-1`，不影响远程调试
`xdebug.var_display_max_depth`
整数，默认`3`，显示数组或对象属性时的最大嵌套深度，最大`1023`，可以用`-1`指代这个最大数

**最大递归保护：**
设定以下配置项：
`xdebug.max_nesting_level`
整数，默认为：`256`，不限设置为`-1`，无限递归的保护机制，当递归调用达到该设定时程序被中断，注意此时不会出现任何错误提示，而是直接退出程序

**函数调用追踪：**
包括对象方法调用，该功能默认是关闭的，开启后将输出调用数据到一个文件，查看文件即可，配置如下（这里值设定为我们最常用的情况以便复制使用，更多见下文的配置说明）：
`xdebug.auto_trace=1`
;开启追踪，默认是关闭的
`xdebug.trace_output_dir="C:\root\xdebug\trace"`
;调用追踪数据输出目录
`xdebug.trace_output_name="yunke.%s.%u"`
;追踪文件的文件名
`xdebug.collect_params=4`
;收集函数参数的形式
`xdebug.collect_return=1`
;是否收集函数返回值
`xdebug.show_mem_delta=1`
;显示内存详情
`xdebug.trace_format=0`
;追踪文件的格式

以上设置将为所有请求产生分析文件，调用追踪分析还可以采用变量触发的方式（可通过浏览器扩展自动完成，这也是推荐方法，见下），也就是在`GET/POST` 或`COOKIE`中设置变量名`XDEBUG_TRACE`，它的值是和以下设置项匹配的密钥，配置如下：
`xdebug.auto_trace=0`
;开启变量触发跟踪时自动跟踪需要关闭
`xdebug.trace_enable_trigger=1`
`xdebug.trace_enable_trigger_value="yunke"`
;这就是上述`XDEBUG_TRACE`变量的值
如访问网址：http://www.test.com/index.php?XDEBUG_TRACE=yunke，即可触发追踪文件的产生
这里"yunke"是一个密钥值，只有和配置文件匹配才能产生分析文件，默认为空字符

**代码覆盖分析：**
该功能可以让我们知道代码执行中哪些行被执行到了，通常用于单元测试，配置如下：
`xdebug.coverage_enable=1`
;代码覆盖分析开启

代码覆盖分析是通过在php代码中调用函数来进行的，过程如下，那将得到一个数组结果：
```
xdebug_start_code_coverage();//开启覆写
…//被分析的一些代码
var_dump(xdebug_get_code_coverage());//得到一个分析结果数组
```


**垃圾回收分析：**
该功能默认是关闭的，开启后将输出垃圾回收分析数据到一个文件，查看文件即可，配置如下（这里值设定为我们最常用的情况以便复制使用，更多见下文的配置说明）：
`xdebug.gc_stats_enable=1`
;开启垃圾回收分析
`xdebug.gc_stats_output_dir="C:\root\xdebug\gc"`
;数据输出目录，默认是`/tmp`
`xdebug.gc_stats_output_name=”gcstats.%s.%u”`
;输出文件名，默认是: `gcstats.%p`




**程序性能分析：**
该功能默认是关闭的，开启后将输出性能分析数据到一个文件，该文件是便于机器解析的文本格式，需要专用软件查看，配置如下（这里值设定为我们最常用的情况以便复制使用，更多见下文的配置说明）：
`xdebug.profiler_enable=1`
;开启性能分析
`xdebug.profiler_output_dir="C:\root\xdebug\profiler"`
;性能分析文件输出目录
`xdebug.profiler_output_name="cachegrind.out.%s.%u"`
;性能分析输出文件名

以上设置将为所有请求产生分析文件，性能分析还可以采用变量触发的方式（可通过浏览器扩展自动完成，这也是推荐方法，见下），也就是在`GET/POST` 或`COOKIE`中设置变量名`XDEBUG_PROFILE`，它的值是和以下设置项匹配的密钥，配置如下：
`xdebug.profiler_enable=0`
;开启变量触发分析时自动分析需要关闭
`xdebug.profiler_enable_trigger=1`
`xdebug.profiler_enable_trigger_value="yunke"`
;这就是上述`XDEBUG_PROFILE`变量的值
即可触发分析文件的产生，如访问网址：http://www.test.com/index.php?XDEBUG_PROFILE=yunke
这里"yunke"是一个密钥值，只有和配置文件匹配才能产生分析文件，默认为空字符

在官网介绍了许多种软件来查看输出的分析文件，其中`WinCacheGrind`在小型程序分析中尚可，大型程序就力不从心了，甚至会出现错误，经云客测试采用`QCacheGrind` 效果最好，它是`KCacheGrind`的Windows版本，这里对该软件做一个简单介绍：
打开后左边有一个“Flat Profile”面板，里面第一列“Incl.”是执行时间，包括内部调用的子程序的时间；Self列是自身消耗的时间，不包括子函数调用；Called列是该函数被调用的次数；Function是函数名；Location列是文件位置；
时间是以微秒为单位1秒=1000000 微秒(μs)，以可以采用百分比的方式

**远程调试：**
我们先看一下远程调试的整个过程，首先由浏览器附带特殊参数打开要调试的脚本，脚本开始运行，此时Xdebug会主动向配置好的调试客户端发起一个连接，并采用DBGp协议和调试客户端交互（调试客户端往往就是我们使用的IDE，如phpstorm，它需要监听调试网络端口），调试客户端通过协议让Xdebug一步一步的执行代码，显示或设置程序中的变量内容，在每一步之间Xdebug将php程序挂起暂停执行，这个“步”又叫断点，调试客户端可以通过设置断点来让程序在该处暂停，对于调试客户端而言这又叫做断点调试，整个过程而言是Xdebug在php引擎里一步一步的执行代码并操作这个过程，她接收调试客户端的指令。
要进行远程调试具体操作如下，首先需要以下配置并重启php：
`xdebug.remote_enable=1`
;远程调试开关
`xdebug.remote_host=localhost`
;远程调试客户端主机地址，也就是IDE所在主机的地址，往往就是localhost
`xdebug.remote_port=9000`
;远程调试端口
配置好后，我们在浏览器中打开脚本，打开时需要发送`GET/POST`变量名`XDEBUG_SESSION_START`或者`COOKIE`变量名`XDEBUG_SESSION`，其值是Xdebug和调试客户端的通信会话id，往往是ide指定的一个特殊字符，如phpstorm指定为“`PHPSTORM`”，该值可以在`xdebug.idekey`配置中设定，默认为空字符串；浏览器附加的参数可以通过浏览器扩展自动完成（见下）；服务器收到请求后依据`xdebug.remote_mode`配置项的设定开始调试过程，该配置项如果是“`req`”那么脚本刚启动时Xdebug就向调试客户端（IDE）发起调试连接，这也是默认值，如果是“`jit`”那么仅在有错误出现时才发起连接，一旦和IDE的连接建立成功那么就采用DBGp协议进行调试交互。
如果服务器是被多个开发者共享的，那么调试客户机会有多个，而`xdebug.remote_host`配置项只能设定一个，此时可以将`xdebug.remote_connect_back`设定为`1`，那将采用从http头中获取的ip来作为调试客户机的地址。
Xdebug在发起调试连接时默认超时时间为200毫秒，该值在`xdebug.remote_timeout`配置项中设置，在本机调试该值足够了，如果是远程的网络主机那么需要加大该值以应对网络延迟。


**浏览器插件辅助：**
如前文所述，触发方式产生分析、追踪文件和开启远程调试都需要在`GET/POST` 或`COOKIE`里面设置特定变量和值，那么我们能否让浏览器代劳呢，这是可以的，这里以开发者使用最多的火狐浏览器为列来说明：
先安装辅助插件，打开地址：https://addons.mozilla.org/en-GB/firefox/addon/xdebug-helper-for-firefox/
你将看到`Xdebug Helper for Firefox` ，点击Add to Firefox，按照提示操作即可
安装完成后需要进行配置：按组合键“`ctrl+shift+a`”打开附加组件面板，找到Xdebug Helper，点击选项，分别输入IDE远程会话id、分析密钥、追踪密钥（对应配置中`xdebug.idekey`、`xdebug.profiler_enable_trigger_value`、`xdebug.trace_enable_trigger_value`），点击保存，此时新开网页的浏览器地址栏中将出现一个爬虫图标，可以选择四种状态：调试、分析、追踪、禁用，前三种种状态下浏览器打开连接时会附带额外的cookie参数，比如我们选择分析，那么刷新连接时将在cookie中添加`XDEBUG_PROFILE`变量，值为前文设定的密钥值，这将让Xdebug产生分析文件，函数追踪和远程调试同理。


### 【常用配置】
全部设置及详细见：https://xdebug.org/docs/all_settings，本文只进行大概介绍
`xdebug.trace_output_dir`
函数调用跟踪数据文件的写入目录，默认为 /tmp，确保可写
`xdebug.trace_output_name`
追踪文件的文件名指引，默认为：`trace.%c`  比如：`yunke.%s.%u` 将输出带路径的脚步名及带微秒的时间，如下：
```
yunke.C__root_test_index_php.1520473784_260486.xt
```

`xdebug.auto_trace`
开启函数调用追踪，布尔值，默认为`0`
`xdebug.collect_assignments`
布尔值，默认为`0`，是否在函数追踪中添加变量赋值
`xdebug.collect_includes`
布尔值，默认为`1`，是否将`include()`, `include_once()`, `require()` or `require_once()` 的文件写入追踪文件
`xdebug.collect_params`
整数，默认`0`，决定函数追踪的参数收集，`0`为不收集，`1`参数的类型和数量，`2`在1基础上加工具提示信息，`3`全变量内容（受到变量输出设置的影响），`4`全变量内容和变量名，`5`php序列化内容没有变量名
`xdebug.collect_return`
布尔值，默认`0`，是否写入函数调用的返回值到追踪文件
`xdebug.show_mem_delta`
整数，默认为`0`，非0值将显示函数调用的内存用量信息
`xdebug.trace_format`
整型，默认`0`，追踪文件的格式，`0`为人类可读格式（时间索引，内存用量等等）`1`机器易读格式  `2`人类可读格式以网页展现
`xdebug.trace_options`
整数，默认为`0`，如果设置为`1`，那么追踪文件采用追加方式，而不是覆盖
`xdebug.var_display_max_children`
整数，默认`128`，显示的数组子元素或对象属性的最大数量，不限制设定为`-1`，远程调试时不受影响
`xdebug.var_display_max_data`
整数，默认`512`，显示字符串的最大长度，不限设置为`-1`，不影响远程调试
`xdebug.var_display_max_depth`
整数，默认`3`，显示数组或对象属性时的最大嵌套深度，最大`1023`，可以用`-1`指代这个最大数
`xdebug.coverage_enable`
布尔，默认为`1`，是否开启代码覆盖分析，实测该设置项无效，开启和得到分析结果需用函数
`xdebug.gc_stats_enable`
布尔值，默认为`0`，是否开启垃圾回收统计分析
`xdebug.gc_stats_output_dir`
垃圾统计分析的写入目录，注意权限
`xdebug.gc_stats_output_name`
垃圾分析文件的文件名，和追踪分析的文件名规则一样
`xdebug.profiler_enable`
整数，默认为`0`，为`1`时将开启性能分析功能
`xdebug.profiler_aggregate`
整数，默认`0`，非`0`时将多个请求的分析数据写入一个文件中以进行跨请求分析
`xdebug.profiler_append`
整数，默认`0`，分析文件是否采用追加模式，文件名的设定对该项有影响
`xdebug.profiler_enable_trigger`
整数，默认为0，采用触发模式开启分析功能，开启它时需要关闭`xdebug.profiler_enable`
`xdebug.profiler_enable_trigger_value`
字符串。默认为“”，触发分析的密钥，配合`xdebug.profiler_enable_trigger`使用
`xdebug.profiler_output_dir`
字符串，默认为`/tmp`，分析文件输出目录
`xdebug.profiler_output_name`
分析文件名称，默认为`cachegrind.out.%p`，见`xdebug.trace_output_name`
`xdebug.extended_info`
整型，默认为`1`，是否强制`php`解析器执行`extended_info`模式
`xdebug.idekey`
字符串，默认:` *complex*`，调试会话id，经测试浏览器发送任意值均可开始调试，所以她并非密钥值，但有些调试客户端可以通过它来判断是否接受调试连接，所以最好统一
`xdebug.remote_addr_header`
默认为空字符串“”，用以指定哪个http头代表调试客户端地址，和`xdebug.remote_connect_back `组合使用
`xdebug.remote_autostart`
布尔值，默认`0`，通常使用特定变量开始远程调试，如果该项被设置为`1`，那么总是开启
`xdebug.remote_connect_back`
布尔值，默认`0`，解决多人调试问题，忽略设定的固定ip，指示服务器依据请求地址链接
`xdebug.remote_cookie_expire_time`
整数。默认`3600`，远程调试cookie超期时间
`xdebug.remote_enable`
布尔值，默认`0`，是否启用远程调试
`xdebug.remote_host`
字符串，默认：`localhost`，远程调试客户端的地址
`xdebug.remote_log`
字符串，默认空，远程调试日志文件名
`xdebug.remote_mode`
字符串，远程调试默认，`req`脚本一启动就链接，`jit`当错误发生时链接
`xdebug.remote_port`
远程调试端主机端口，默认`9000`
`xdebug.remote_timeout`
整数，默认`200`，单位毫秒，等待调试链接的时间
`xdebug.default_enable`
布尔值，`1`或`0`，开启以xdebug的方式进行错误提示，默认开启
`xdebug.max_nesting_level`
整数，默认为：`256`，无限递归的保护机制，当递归调用达到该设定时程序被中断
`xdebug.max_stack_frames`
整数，默认值`-1`，设定错误提示时堆栈中有多少个帧被显示
`xdebug.scream`
布尔值，默认为`0`，是否禁用“`@`”，以便错误被强制显示


xdebug扩展开启后具备的函数：
当扩展被加载后，php脚本中可以使用以下函数：
（这里只列出部分，全部请见https://xdebug.org/docs/all_functions）
```
string xdebug_call_class( [int $depth = 1] )
```
显示调用类
```
string xdebug_call_file( [int $depth = 1] )
```
显示调用文件
```
string xdebug_call_function( [int $depth = 1] )
```
返回调用函数
```
int xdebug_call_line( [int $depth = 1] )
```
返回调用行
```
void xdebug_disable()
```
禁用堆栈跟踪
```
void xdebug_enable()
```
开启堆栈跟踪
```
bool xdebug_is_enabled()
```
检查堆栈跟踪是否被开启
```
string xdebug_get_collected_errors( [int clean] )
```
从错误集缓冲中返回所有错误信息
```
array xdebug_get_headers()
```
返回header() 函数设置的所有头信息
```
int xdebug_memory_usage()
```
返回内存使用量
```
int xdebug_peak_memory_usage()
```
返回到目前为止脚本使用过的最大内存用量
```
void xdebug_start_error_collection()
```
收集错误并禁止显示
```
void xdebug_stop_error_collection()
```
停止错误记录，并从缓冲中收集
```
float xdebug_time_index()
```
返回当前点的执行时间，单位秒

>xdebug配置解释部分转载自 [php调试工具Xdebug使用教程](https://blog.csdn.net/u011474028/article/details/79571909)

三、配置PhpStorm
---

打开PhpStorm 按 `ctrl + alt + s`打开设置界面，配置如下，Debug port 与在`php.ini`内配置的`xdebug.remote_port`对应；点击 apply 保存再点 ok 关闭

![image.png](https://upload-images.jianshu.io/upload_images/2305018-333cd43f1de6e2d0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来配置服务器，在PhpStorm右上角下拉选择   “Edit  Configurations”
![image.png](https://upload-images.jianshu.io/upload_images/2305018-456f4a186b2c9050.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如果没有服务器配置，需要新建一个；点击左上角的加号新建，新建类型我这里选择的是`PHP Web Page` ，也可以选择`PHP application`
![image.png](https://upload-images.jianshu.io/upload_images/2305018-038b5ee22278cbee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

再点击上图所示右侧的 server  配置服务器，Host根据实际情况填写，本地的话可以填`localhost`

![image.png](https://upload-images.jianshu.io/upload_images/2305018-8e073822e56c1829.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

下面点击行号的右边在PHP代码中打断点
![image.png](https://upload-images.jianshu.io/upload_images/2305018-54e40c49cbc5c781.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

首先点击PhpStorm右上角的电话图标，再点击虫子图标，这时会打开浏览器

![image.png](https://upload-images.jianshu.io/upload_images/2305018-48a524c1fd8a1295.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到已经自动在浏览器中打开了刚才配置的地址，但是这不是我们想要调试的页面，现在可以将红框内查询字符串前面的地址改成所要调试的地址
![image.png](https://upload-images.jianshu.io/upload_images/2305018-0957634db7810496.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

保留问号后的查询字符串，修改地址后按回车
![修改地址](https://upload-images.jianshu.io/upload_images/2305018-96513e18e2653c69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

PhpStorm就会自动显示断点位置的调试信息了
![image.png](https://upload-images.jianshu.io/upload_images/2305018-d16702495220e639.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

至此，PhpStorm + Xdebug 断点调试 配置完毕


四、浏览器插件调试
---
除了上面所讲的利用`XDEBUG_SESSION_START` 地址栏`query_string`传值的方式，我们还可以利用`COOKIE`传递`XDEBUG_SESSION`，来使得PhpStorm能够检测到我们在浏览器发起的调试请求。`XDEBUG_SESSION`的值就是`idekey`的值，可以通过手动修改头信息的方式传递`COOKIE`，但是这样就比较麻烦，这时我们可以借助浏览器插件来帮我们设置`COOKIE`。

以Chrome为例，我们可以下载Chrome插件：` Xdebug helper ` ，建议搭个梯子到Chrome官方插件商店下载。

![xdebug helper](https://upload-images.jianshu.io/upload_images/2305018-f2877a6a5e648203.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

添加成功之后鼠标右键点击浏览器右上角插件栏上的虫子，进入“选项”将`IDEKEY`设置成`PHPSTORM`并保存，就可以愉快的开始调试了

![xdebug helper](https://upload-images.jianshu.io/upload_images/2305018-656ad2a198a73d7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击浏览器右上角的灰色虫子，选择debug选项，虫子变为绿色就开启了；

![xdebug helper](https://upload-images.jianshu.io/upload_images/2305018-64638f49ba42e28b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来还要开启PhpStorm监听，点击PhpStorm右上角的电话按钮，变成如下样式就代表开启了监听。与之前不同的是，使用浏览器插件调试时不需要点击PhpStorm右上角的虫子获取ID。

![PhpStorm](https://upload-images.jianshu.io/upload_images/2305018-d33d750120f03e78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

打断点后运行，就可以看到调试信息了
![断点调试](https://upload-images.jianshu.io/upload_images/2305018-c2ab28a4dc77f393.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)








