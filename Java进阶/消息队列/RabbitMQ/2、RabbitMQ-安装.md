# 一、安装RabbitMQ
* 1、下载
````
1、官网下载地址：https://www.rabbitmq.com/download.html(opens new window)
2、GitHub：https://github.com/rabbitmq/rabbitmq-server/releases/tag/v3.8.8(opens new window)
3、加载下载：https://packagecloud.io/rabbitmq/rabbitmq-server/packages/el/7/rabbitmq-server-3.8.8-1.el7.noarch.rpm(opens new window)
这里我们选择的版本号（注意这两版本要求）
rabbitmq-server-3.8.8-1.el7.noarch.rpm

安装erlang环境
erlang-21.3.8.21-1.el7.x86_64.rpm
官网：https://www.erlang-solutions.com/downloads/
加速：https://packagecloud.io/rabbitmq/erlang/packages/el/7/erlang-21.3.8.21-1.el7.x86_64.rpm(opens new window)
Red Hat 8, CentOS 8 和 modern Fedora 版本，把 “el7” 替换成 “el8”
````
* 2、安装
````
上传到 /usr/local/software 目录下(如果没有 software 需要自己创建)
rpm -ivh erlang-21.3.8.21-1.el7.x86_64.rpm
yum install socat -y
rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm
````
* 3、启动
````
# 添加开机启动
chkconfig rabbitmq-server on
# 启动服务
/sbin/service rabbitmq-server start
# 查看服务状态
/sbin/service rabbitmq-server status
# 停止服务
/sbin/service rabbitmq-server stop
# 重启服务
/sbin/service rabbitmq-server restart
````
# 二、Web管理界面及授权操作
* 1、安装
````
默认情况下，是没有安装web端的客户端插件，需要安装才可以生效
rabbitmq-plugins enable rabbitmq_management

安装完毕以后，启动服务即可
/sbin/service rabbitmq-server start

访问 http://10.68.63.68:15672 ，用默认账号密码(guest)登录，出现权限问题
默认情况只能在 localhost 本机下访问，所以需要添加一个远程登录的用户
````
* 2、添加用户
````
# 创建账号和密码
rabbitmqctl add_user admin 123456

# 设置用户角色
rabbitmqctl set_user_tags admin administrator

# 为用户添加资源权限
# set_permissions [-p <vhostpath>] <user> <conf> <write> <read>
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
# 添加配置、写、读权限
````
* 3、用户级别：
````
administrator：可以登录控制台、查看所有信息、可以对 rabbitmq 进行管理
monitoring：监控者 登录控制台，查看所有信息
policymaker：策略制定者 登录控制台，指定策略
managment：普通管理员 登录控制台
再次登录，用 admin 用户
````
* 4、重置命令
````
关闭应用的命令为：rabbitmqctl stop_app
清除的命令为：rabbitmqctl reset
重新启动命令为：rabbitmqctl start_app
````
# 三、Docker 安装
````
官网：https://registry.hub.docker.com/_/rabbitmq/
docker run -id --name myrabbit -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=123456 -p 15672:15672 rabbitmq:3-management
````