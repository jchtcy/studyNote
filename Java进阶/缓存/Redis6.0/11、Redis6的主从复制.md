# 一、Redis主从复制
* 1、定义
````
主机数据更新后根据配置和策略， 自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主
````
* 2、作用
````
读写分离，性能扩展
容灾快速恢复
````
# 二、主从复制步骤
````
拷贝多个redis.conf文件include(写绝对路径)
开启daemonize yes
Pid文件名字pidfile
指定端口port
Log文件名字
dump.rdb名字dbfilename
Appendonly 关掉或者换名字
````
* 1、 新建redis6379.conf，填写以下内容
````
include /myredis/redis.conf
pidfile /var/run/redis_6379.pid
port 6379
dbfilename dump6379.rdb
````
* 2、 新建redis6380.conf，填写以下内容
````
include /myredis/redis.conf
pidfile /var/run/redis_6380.pid
port 6380
dbfilename dump6380.rdb
````
* 3、 新建redis6381.conf，填写以下内容
````
include /myredis/redis.conf
pidfile /var/run/redis_6381.pid
port 6381
dbfilename dump6381.rdb
````
* 4、启动三台Redis服务器
````
redis-server redis6379.conf
redis-server redis6380.conf
redis-server redis6381.conf
````
* 5、查看系统进程，看看三台服务器是否启动
````
查看三台主机运行情况
info replication
````
* 6、配从(库)不配主(库)
````
slaveof ip port
成为某个实例的从服务器

在6380和6381上执行: slaveof 127.0.0.1 6379
在主机上写，在从机上可以读取数据
在从机上写数据报错

从机重启需重设：slaveof 127.0.0.1 6379
可以将配置增加到文件中。永久生效。
````
# 三、复制原理
````
Slave启动成功连接到master后会发送一个sync命令
Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步
但是只要是重新连接master,一次完全同步(全量复制)将被自动执行
````
# 四、常用三招
* 1、一主二仆
````
切入点问题？slave1、slave2是从头开始复制还是从切入点开始复制?比如从k4进来，那之前的k1,k2,k3是否也可以复制？
slave1、slave2是从头开始复制。之前的k1,k2,k3也可以复制

从机是否可以写？set可否？
从机不能写数据,只能读数据,所以不能set

主机shutdown后情况如何？从机是上位还是原地待命？
主机挂掉,从服务器不做操作。

主机又回来了后，主机新增记录，从机还能否顺利复制？
主机回来后,主机仍然是主机,从机仍然是从机,可以顺利复制

其中一台从机down后情况如何？依照原有它能跟上大部队吗？
从机down后,从新连接会进行一个全量复制
````
* 2、薪火相传
````
上一个Slave可以是下一个slave的Master，Slave同样可以接收其他 slaves的连接和同步请求，那么该slave作为了链条中下一个的master, 可以有效减轻master的写压力,去中心化降低风险。

用 slaveof ip port
中途变更转向:会清除之前的数据，重新建立拷贝最新的
风险是一旦某个slave宕机，后面的slave都没法备份
主机挂了，从机还是从机，无法写数据了
````
* 3、反客为主
````
当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改。

用 slaveof no one 将从机变为主机。
````
# 五、哨兵模式(sentinel)
* 1、定义
````
反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库
````
* 2、操作步骤
````
1、调整为一主二仆模式，6379带着6380、6381
2、创建sentinel.conf文件
自定义的/myredis目录下新建sentinel.conf文件，名字绝不能错
3、配置哨兵,填写内容
sentinel monitor mymaster 127.0.0.1 6379 1
其中mymaster为监控对象起的服务器名称， 1 为至少有多少个哨兵同意迁移的数量。
4、启动哨兵
/usr/local/bin
redis做压测可以用自带的redis-benchmark工具
执行redis-sentinel /myredis/sentinel.conf
5、当主机挂掉，从机选举中产生新的主机
(大概10秒左右可以看到哨兵窗口日志，切换了新的主机)
哪个从机会被选举为主机呢？根据优先级别：slave-priority
原主机重启后会变为从机
6、复制延时
由于所有的写操作都是先在Master上操作，然后同步更新到Slave上，所以从Master同步到Slave机器有一定的延迟，当系统很繁忙的时候，延迟问题会更加严重，Slave机器数量的增加也会使这个问题更加严重。
````
* 3、故障恢复
![本地路径](img/)
````
优先级在redis.conf中默认：replica-priority100，值越小优先级越高
偏移量是指获得原主机数据最全的
每个redis实例启动后都会随机生成一个40位的runid
````