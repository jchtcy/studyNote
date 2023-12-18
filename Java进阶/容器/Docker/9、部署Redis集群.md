````
1、先移除之前的容器
root@jch-virtual-machine:/# docker rm -f $(docker ps -aq)

2、创建redis的网卡
root@jch-virtual-machine:/# docker network create redis --subnet 172.38.0.0/16
14ea9f6beb51d4558733b6dc068abe94d57c2ff0af5bf327fa1954bd864a7a3a
root@jch-virtual-machine:/# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
d47ee459755b   bridge    bridge    local
98a1369567ea   host      host      local
a88fae429570   mynet     bridge    local
d05180111537   none      null      local
14ea9f6beb51   redis     bridge    local

3、通过脚本去写配置文件并启动
for port in $(seq 1 6); do mkdir -p /mydata/redis/node-${port}/conf; touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
 docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} -v /mydata/redis/node-${port}/data:/data -v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf -d --net redis --ip 172.38.0.1${port} redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf; done

ff9b43a9e10ca8cbeeba0bb27ed4e49fb9277f400ba1ee38d1354a4af4aceee6
a7acc469567cd676c9796aecdb63f71cc7f661508aaec01d093d0d3b9bc7ae58
687ab9a7674eeb031ac43873da2445d301f20160e9690ad1e52bffd5b4420935
801d7fb47c01ff888ce92843a422d620b26d0686ab8629a5568d26d1cbbc7a8b
9a8368d8791c8d15efa8e1b61ad06000d1ea7b33e23d4c0333f5cf7766589c6c
e390b3dca06d0c7717daef2f0c6eb4b3f444fcc7c549b62903ac266529b43ca6

4、查看容器
root@jch-virtual-machine:/mydata/redis# docker ps
CONTAINER ID   IMAGE                    COMMAND                   CREATED          STATUS          PORTS                                                                                      NAMES
e390b3dca06d   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   14 seconds ago   Up 8 seconds    0.0.0.0:6376->6379/tcp, :::6376->6379/tcp, 0.0.0.0:16376->16379/tcp, :::16376->16379/tcp   redis-6
9a8368d8791c   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   19 seconds ago   Up 14 seconds   0.0.0.0:6375->6379/tcp, :::6375->6379/tcp, 0.0.0.0:16375->16379/tcp, :::16375->16379/tcp   redis-5
801d7fb47c01   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   24 seconds ago   Up 19 seconds   0.0.0.0:6374->6379/tcp, :::6374->6379/tcp, 0.0.0.0:16374->16379/tcp, :::16374->16379/tcp   redis-4
687ab9a7674e   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   28 seconds ago   Up 24 seconds   0.0.0.0:6373->6379/tcp, :::6373->6379/tcp, 0.0.0.0:16373->16379/tcp, :::16373->16379/tcp   redis-3
a7acc469567c   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   32 seconds ago   Up 28 seconds   0.0.0.0:6372->6379/tcp, :::6372->6379/tcp, 0.0.0.0:16372->16379/tcp, :::16372->16379/tcp   redis-2
ff9b43a9e10c   redis:5.0.9-alpine3.11   "docker-entrypoint.s…"   36 seconds ago   Up 32 seconds   0.0.0.0:6371->6379/tcp, :::6371->6379/tcp, 0.0.0.0:16371->16379/tcp, :::16371->16379/tcp   redis-

5、进入其中一个容器
root@jch-virtual-machine:/home/jch# docker exec -it redis-1 /bin/sh 

6、创建集群
/data # redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13
:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379  --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
Adding replica 172.38.0.14:6379 to 172.38.0.13:6379
M: a9a2a0b84591c6916848291083cb139ea1f78fbb 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
M: b10eba0f6cd57e8a52dd401037865ea97db1eca1 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
M: 0693e17443fc5bb187a62bf556675f70f0bf1b0e 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
S: f3ec21cb4a0f2f39d116c58f4dadd2216d909980 172.38.0.14:6379
   replicates 0693e17443fc5bb187a62bf556675f70f0bf1b0e
S: d5b3310d9ce66db4e344dcd3e1c81378d2141cdf 172.38.0.15:6379
   replicates a9a2a0b84591c6916848291083cb139ea1f78fbb
S: 63439156ed2aee4d94d4464871a54afae6ed1ee3 172.38.0.16:6379
   replicates b10eba0f6cd57e8a52dd401037865ea97db1eca1
Can I set the above configuration? (type 'yes' to accept): 

7、进入redis服务
/data # redis-cli -c

8、查看集群
127.0.0.1:6379> cluster info
cluster_state:fail
cluster_slots_assigned:0
cluster_slots_ok:0
cluster_slots_pfail:0
cluster_slots_fail:0
cluster_known_nodes:1
cluster_size:0
cluster_current_epoch:0
cluster_my_epoch:0
cluster_stats_messages_sent:0
cluster_stats_messages_received:0
````