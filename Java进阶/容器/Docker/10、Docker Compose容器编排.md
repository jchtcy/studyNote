# 一、定义
````
Docker-Compose 是 Docker 官方的开源项目，负责实现对Docker容器集群的快速编排。

Docker-Compose可以管理多个Docker容器组成一个应用。需要定义一个yaml格式的配置文件 docker-compose.yml，配置好多个容器之间的调用关系，然后只需要一个命令就能同时启动/关闭这些容器。

Docker建议我们每个容器中只运行一个服务，因为Docker容器本身占用资源极少，所以最好是将每个服务单独的分割开来。但是如果我们需要同时部署多个服务，每个服务单独构建镜像构建容器就会比较麻烦。所以 Docker 官方推出了 docker-compose 多服务部署的工具。

Compose允许用户通过一个单独的 docker-compose.yml 模板文件来定义一组相关联的应用容器为一个项目（project）。可以很容易的用一个配置文件定义一个多容器的应用，然后使用一条指令安装这个应用的所有依赖，完成构建。
````
* 1、核心概念
````
服务(service): 一个个应用容器实例
工程(project): 由一组关联的应用容器组成的一个完整业务单元，在docker-compose.yml中定义
````
* 2、Compose使用的三个步骤
````
1、编写 Dockerfile 定义各个应用容器，并构建出对应的镜像文件
2、编写 docker-compose.yml，定义一个完整的业务单元，安排好整体应用中的各个容器服务
3、执行 docker-compose up 命令，其创建并运行整个应用程序，完成一键部署上线
````
# 二、安装Docker-Compose
````
1、Docker-Compose的版本需要和Docker引擎版本对应

2、从github下载 docker-compose-linux-x86_64

3、移动文件
root@jch-virtual-machine:/# mv /mnt/hgfs/share/docker-compose-linux-x86_64 /usr/local/bin/docker-compose

4、添加权限
root@jch-virtual-machine:/# chmod +x /usr/local/bin/docker-compose

5、验证
root@jch-virtual-machine:/# docker-compose version
Docker Compose version v2.23.2

6、卸载Compose：直接删除 usr/local/bin/docker-compose文件即可
````
# 三、常用命令
````
命令	说明
docker-compose -h	查看帮助
docker-compose up	启动所有docker-compose服务
docker-compose up -d	启动所有docker-compose服务并后台运行
docker-compose down	停止并删除容器、网络、卷、镜像
docker-compose exec yml里面的服务id	进入容器实例内部 docker-compose exec docker-compose.yml文件中写的服务id /bin/bash
docker-compose ps	展示当前docker-compose编排过的运行的所有容器
docker-compose top	展示当前docker-compose编排过的容器进程
docker-compose logs yml里面的服务id	查看容器输出日志
docker-compose config	检查配置
docker-compose config -q	检查当前路径下的compose.yml文件配置，有问题才有输出
docker-compose restart	重启服务
docker-compose start	启动服务
docker-compose stop	停止服务
````
# 四、使用Compose
* 1、编写docker-compose.yml文件
````
#compose版本
version: "3"  
 
#微服务项目	
services:
  microService:
#微服务镜像  
    image: zzyy_docker:1.6
    container_name: ms01
    ports:
      - "6001:6001"
#数据卷
    volumes:
      - /app/microService:/data
    networks: 
      - atguigu_net 
    depends_on: 
      - redis
      - mysql
      
#redis服务
  redis:
    image: redis:6.0.8
    ports:
      - "6379:6379"
    volumes:
      - /app/redis/redis.conf:/etc/redis/redis.conf
      - /app/redis/data:/data
    networks: 
      - atguigu_net
    command: redis-server /etc/redis/redis.conf
 
 #mysql服务
  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: '123456'
      MYSQL_ALLOW_EMPTY_PASSWORD: 'no'
      MYSQL_DATABASE: 'db2021'
      MYSQL_USER: 'zzyy'
      MYSQL_PASSWORD: 'zzyy123'
    ports:
       - "3306:3306"
    volumes:
       - /app/mysql/db:/var/lib/mysql
       - /app/mysql/conf/my.cnf:/etc/my.cnf
       - /app/mysql/init:/docker-entrypoint-initdb.d
    networks:
      - atguigu_net
    command: --default-authentication-plugin=mysql_native_password #解决外部无法访问
 
 #创建自定义网络
networks: 
   atguigu_net: 
````
* 2、修改微服务项目，将映射的ip修改为服务名
````
​将SpringBoot项目的配置文件中mysql和redis的ip更改为项目名。之后打包成jar包，在docker中编写Dockerfile文件，构建镜像。
````
* 3、检查当前目录下compose.yml文件是否有语法错误
````
docker compose config -q
````
* 4、启动所有docker-compose服务并后台运行
````
docker compose up -d
````