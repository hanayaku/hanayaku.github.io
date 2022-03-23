---
layout: post
title:  "cgi、fast-cgi、php-cgi、php-fpm"
summary: "关于cgi、fast-cgi、php-cgi、php-fpm的理解"
author: hana
date: '2022-03-12'
category: ['php']
tags: jekyll
thumbnail: /assets/img/posts/php.jpg
keywords: learn
usemathjax: false
permalink: /blog/cgi/
---


> 个人理解：首先来说过程：web server，比如nginx，是内容分发者。如果有个请求是/index.php，根据配置文件，nginx找到PHP解析器来处理，PHP解析器就是PHP-CGI啦~PHP解析器解析php,ini文件，初始化执行环境，然后处理请求，再以CGI规定的格式返回处理后的结果，提出进程，web server再把结果返回给浏览器。通过CGI这个协议呢，可以看出PHP执行的很累啊，来一个请求就初始化、解析、处理、返回，那么就用fastcgi吧！他比CGI厉害哦，就像是车票大厅，多开了几个购票窗口，一次初始化，产生很多个worker，后面的请求就交给各个worker去处理啦。对了，CGI 和 fastcgi都是协议，就像是购票员脑子里的一到业务流程，它们必须要存在于一个载体中，通过这个载体去实现。此时，PHP-CGI，是实现了CGI的程序，PHP-FPM是实现了fastcgi的程序。注意啦注意啦，问题一和问题二的提问都是错误的。首先，cgi、fastcgi只是一个协议，php-cgi和php-fpm是程序，他们管不了进程。总结一下，如果要修改web配置，则前往web server(apache、nginx)去修改；如果要修改php解析方面的配置，则前往php-fpm和php.ini啦
> 

### 一、认识
1. cgi：Common Gateway Interface 公共网关接口 ，Web服务器运行时外部程序的规范。是外部扩展应用程序与 Web 服务器交互的一个标准接口。

2. 按照这个规范有什么好处？答：按CGI编写的程序可以拓展服务器功能。这种程序叫做“CGI应用程序”。根据CGI标准，编写外部扩展应用程序，可以对客户端浏览器输入的数据进行处理，完成客户端与服务器的交互操作。CGI规范定义了Web服务器如何向扩展应用程序发送消息，在收到扩展应用程序的信息后又如何进行处理等内容。对于许多静态的HTML网页无法实现的功能，通过 CGI可以实现，比如表单的处理、对数据库的访问、搜索引擎、基于Web的数据库访问等等。详见：https://baike.baidu.com/item/CGI/607810

3. fast-cgi：https://zhuanlan.zhihu.com/p/295110834
fast-cgi 是cgi的升级版本，FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一 次。PHP使用PHP-FPM(FastCGI Process Manager)，全称PHP FastCGI进程管理器进行管理。

4. php-fpm PHP-FPM(FastCGI Process Manager：FastCGI进程管理器)是一个PHPFastCGI管理器PHP5.3.3已经集成php-fpm了，不再是第三方的包了。PHP-FPM提供了更好的PHP进程管理方式，可以有效控制内存和进程、可以平滑重载PHP配置，比spawn-fcgi具有更多优点，所以被PHP官方收录了。在./configure的时候带 –enable-fpm参数即可开启PHP-FPM。

5. 使用PHP-FPM来控制PHP-CGI的FastCGI进程
/usr/local/php/sbin/php-fpm{start|stop|quit|restart|reload|logrotate}
--start 启动php的fastcgi进程
--stop 强制终止php的fastcgi进程
--quit 平滑终止php的fastcgi进程
--restart 重启php的fastcgi进程
--reload 重新平滑加载php的php.ini
--logrotate 重新启用log文件

6. SAPI全称是Server Application Programming Interface,即服务器应用编程接口，实质上就是定义了一个统一的接口，它的核心就是一个结构体sapi_module_struct。SAPI提供给了外部应用跟php通信的管道，这个外部应用包括不限于Apache，httpd，liunx终端等，sapi通俗的讲就是php-cgi,php-cli,mod_php等，php就是php内核。PHP架构图如下：
![6fe849c4af990bcfed80e8e032caec2c.png](en-resource://database/3464:1)
7. FastCGI的工作原理：
* Web Server启动时载入FastCGI进程管理器(IIS ISAPI或Apache Module)
* FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
* 当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
* FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。在上述情况中，你可以想象CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的 好处是，持续数据库连接(Persistent database connection)可以工作。


### 三、总结
* PHP-CGI和PHP-FPM是程序
* CGI和Fastcgi都是协议，非编程层面的概念；他们集成到PHP中分别成了：PHP-CGI和PHP-FPM
* 这些概念都放在网络中的哪个地方呢？请求---->PHP-CGI/PHP-FPM ----->PHP脚本
* 总的来说，cgi是一个web服务器和应用程序之间沟通的协议，fastcgi让cgi变得更快很好；php-cgi是php的脚本解析器，他遵循了cgi，可以用来作为web服务器和PHP应用程序沟通的桥梁(当然，按照这个思路，会有c++-cgi，python-cgi等等)；fastcgi是多线程的，需要有东西管理这些线程啊，所以有了php-fpm，php-fpm是适用于php的fastcgi进程管理器(当然，按照这个思路，会有python-fpm等等)。

### 四、其他

#### PHP运行模式
刷到过面试问：fastCGI有几种工作模式？直接上网搜没搜到答案，感觉有点奇怪，这是在问PHP的运行模式还是问fastCGI呢？fastCGI的工作模式应该是问基于fastCGI这个协议下PHP的工作模式。以某种模式运行是基于对应协议来说的，而不是基于某个为了实现这个协议的程序，所以不能一下子联系到php-cgi和php-fpm。

php运行模式有cli命令行模式和web模式，其中在web中运行又分三种运行模式，基于cgi协议的运行模式，基于fastcgi的运行模式以及php_mode模式。

还有以下三种模式：
* ISAPI运行模式
基于IIS应用服务器ISAPI即Internet Server Application Program Interface，是微软提供的一套面向Internet服务的API接口，一个ISAPI的DLL，可以在被用户请求激活后长驻内存，等待用户的另一个请求，还可以在一个DLL里设置多个用户请求处理函数，此外，ISAPI的DLL应用程序和WWW服务器处于同一个进程中，效率要显著高于CGI。（由于微软的排他性，只能运行于windows环境)

* APACHE2HANDLER
基于APACHE应用服务器PHP作为Apache模块，Apache服务器在系统启动后，预先生成多个进程副本驻留在内存中，一旦有请求出现，就立即使用这些空余的子进程进行处理，这样就不存在生成子进程造成的延迟了。这些服务器副本在处理完一次HTTP请求之后并不立即退出，而是停留在计算机中等待下次请求。对于客户浏览器的请求反应更快，性能较高。

* APACHE模块的DLL运行模式
基于APACHE应用服务器此运行模式是我们以前在windows环境下使用apache服务器经常使用的，而在模块化(DLL)中，PHP是与Web服务器一起启动并运行的。（是apache在CGI的基础上进行的一种扩展，加快PHP的运行效率）

#### 配置
如果在控制台运行脚本，server api的参数会显示：Command Line Interface；如果是配置了cgi运行或者fastcgi运行会在server api那里显示CGI/FastCGI

1. 以模块的方式运行，在Apache的配置文件里添加如下配置：
```
LoadModule php5_module "C:/php5/php5apache2_2.dll" //大约line 127
PHPinidir "C:/php5/php.ini"
//修改配置
DirectoryIndex index.html index.php//追加index.php
AddType application/x-httpd-php .php //line 408左右添加
```

2. cgi的运行方式，在php.ini修改配置
```
cgi.force_redirect = 0 //本来是 1 并且去掉注释符号；
```
修改apache的配置，去掉原来的模块配置
```
AddType application/x-httpd-php .php
LoadModule php5_module "C:/php5/php5apache2_2.dll"
PHPinidir "C:/php5/php.ini"
```
加入以下配置
```
AddHandler cgi-script .cgi // line 396
```
然后我们在目录C:/Program Files/Apache Software Foundation/Apache2.2/cgi-bin新建一个cgi文件test.cgi编写如下代码：
```
#!c:/php5/php-cgi.exe
<?
php php phpinfo();
?>

```
最后我们访问http://localhost/cgi-bin/test.cgi，出现Server API 为 CGI/FastCGI就说明配置成功。


如果同时打开多个则会有很多php-cgi.exe,并且在执行完成之后消失：
![](http://cdn-zyh.littletrue.cn/cgi%E8%BF%90%E8%A1%8C.jpg)

3. fastcgi的运行方式

PHP的fastcig方式运行，首先需要去下载fastcgi模块，默认是没有带这个模块的，而cgi是自带的；下载地址http://httpd.apache.org/mod_fcgid/；解压复制其中的mod_fcgid.so和mod_fcgid.pdb，接下来做如下的配置：
```
LoadModule fcgid_module modules/mod_fcgid.so // line 128追加
FcgidInitialEnv PHPRC "c:/php5" //php配置文件 line 129追加
AddHandler fcgid-script .php //添加句柄 即后缀 什么样的文件需要fastcgi解释 line 407追加
FcgidWrapper "c:/php5/php-cgi.exe" .php //解释器路径 line 408
Options Indexes FollowSymLinks ExecCGI //line 221 追加 ExecCGI 意思是目录允许执行CGI脚本

```

出现一下内容说明配置成功：
![](http://cdn-zyh.littletrue.cn/20220310103545.png)
![](http://cdn-zyh.littletrue.cn/20220310103626.png)

### Zend
Zend是PHP解析器的核心实现



