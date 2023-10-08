# 一、linux安装Nginx
* 1、安装 pcre 依赖
````
进入 usr/src 目录下，直接向xshell中拖入下载好的 pcre-8.37.tar.gz 压缩包

使用解压命令 
tar –xvf pcre-8.37.tar.gz

进入解压后的 pcre-8.37 文件夹中
cd pcre-8.37/

并使用 ./configure 命令

先编译 再安装
make && make install

查看版本号
pcre-config --version
````
* 2、安装 openssl 、zlib 、 gcc 依赖
````
安装 openssl 、zlib 、 gcc 依赖

yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel
````
* 3、同理安装nginx，
````
进入 usr/src 目录下，直接向xshell中拖入下载好的 nginx-1.12.2.tar.gz 压缩包

使用解压命令 
tar –xvf nginx-1.12.2.tar.gz

进入解压后的 nginx-1.12.2 文件夹中
cd nginx-1.12.2/

并使用 ./configure 命令

先编译 再安装
make && make install

进入目录 /usr/local/nginx/sbin/nginx 启动服务
./nginx
````
* 4、检查是否安装成功
````
在浏览器中输入linux地址，如http://192.168.xx.xxx/

如果没有显示页面，可以关闭防火墙：
    关闭防火墙：systemctl stop firewalld
    开启防火墙：systemctl start firewalld
    查看防火墙状态：systemctl status firewalld

为了让防火墙不拦截Nginx的端口，可以进行如下设置：
    查看开放的端口号：firewall-cmd --list-all
    设置开放的端口号：firewall-cmd --add-port=80/tcp --permanent
    重启防火墙：firewall-cmd --reload
````
# 二、Nginx常用命令
````
进入nginx目录中，nginx命令必须在nginx的sbin目录下进行使用：

cd /usr/local/nginx/sbin

查看版本号
./nginx -v

启动nginx
./nginx

查看是否启动
ps -ef | grep nginx

停止nginx
./nginx -s stop

重新加载nginx(修改配置文件后执行)
./nginx -s reload
````
# 三、Nginx配置文件
* 1、nginx配置文件位置
````
cd /usr/local/nginx/conf/nginx.conf
````
* 2、配置文件的内容
````
全局块：
从配置文件开始到 events 块之间的内容，主要会设置一些影响 nginx 服务器整体运行的配置指令，主要包括配置运行 Nginx 服务器的用户（组）、允许生成的worker process 数，进程 PID 存放路径、日志存放路径和类型以及配置文件的引入等。
例如：worker_processes 值越大，可以支持的并发处理量也越多

events块：
events 块涉及的指令主要影响 Nginx 服务器与用户的网络连接，常用的设置包括是否开启对多 work process 下的网络连接进行序列化，是否允许同时接收多个网络连接，选取哪种事件驱动模型来处理连接请求，每个 word process 可以同时支持的最大连接数等。

http块：
配置中最频繁的部分，代理、缓存和日志定义等绝大多数功能和第三方模块的配置都在这里，需要注意的是：http 块也可以包括 http 全局块、server 块。
    http 全局块
    http 全局块配置的指令包括文件引入、MIME-TYPE 定义、日志自定义、连接超时时间、单链接请求数上限等。

    server 块
    每个 http 块可以包括多个 server 块，而每个 server 块就相当于一个虚拟主机
    每个 server 块也分为全局 server 块，以及可以同时包含多个 locaton 块
````