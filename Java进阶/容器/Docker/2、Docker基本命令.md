# 一、Docker的常用命令
## 1、帮助命令
````
docker version  # docker版本信息
docker info     # 系统级别的信息，包括镜像和容器的数量
docker 命令 --help 

# 官方文档
https://docs.docker.com/reference/
````
## 2、镜像命令
* 1、docker images 查看所有本地主机上的镜像
````
root@jch-virtual-machine:/home/jch# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    9c7a54a9a43c   7 months ago   13.3kB


# 解释
REPOSITORY      # 镜像的仓库
TAG             # 镜像的标签
IMAGE ID        # 镜像的ID
CREATED         # 镜像的创建时间
SIZE            # 镜像的大小
 
# 可选项
--all , -a      # 列出所有镜像
--quiet , -q    # 只显示镜像的id
````
* 2、docker search 查找镜像
````
docker search mysql #搜索MySQL镜像

root@jch-virtual-machine:/home/jch# docker search mysql
NAME                            DESCRIPTION                                      STARS     OFFICIAL   AUTOMATED
mysql                           MySQL is a widely used, open-source relation…   14685     [OK]       
mariadb                         MariaDB Server is a high performing open sou…   5600      [OK]       

# 可选项
--filter=STARS=3000     # 搜素出来的镜像就是STARS大于3000的

root@jch-virtual-machine:/home/jch# docker search mysql --filter=stars=3000
NAME      DESCRIPTION                                      STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   14685     [OK]       
mariadb   MariaDB Server is a high performing open sou…   5600      [OK]       
````
* 3、docker pull 下拉镜像
````
# 下载镜像，docker pull 镜像名[:tag]
root@jch-virtual-machine:/home/jch# docker pull mysql
Using default tag: latest           #如果不写tag, 默认就是latest
latest: Pulling from library/mysql
72a69066d2fe: Pull complete         # 分层下载，dockerimages的核心，联合文件系统
93619dbc5b36: Pull complete 
99da31dd6142: Pull complete 
626033c43d70: Pull complete 
37d5d7efb64e: Pull complete 
ac563158d721: Pull complete 
d2ba16033dad: Pull complete 
688ba7d5c01a: Pull complete 
00e060b6d11d: Pull complete 
1c04857f594f: Pull complete 
4d7cfa90e6ea: Pull complete 
e0431212d27d: Pull complete 
Digest: sha256:e9027fe4d91c0153429607251656806cc784e914937271037f7738bd5b8e7709 # 签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest  # 真实地址
# docker pull mysql 等价于 docker pull docker.io/library/mysql:latest
````
````
# 指定版本下载
root@jch-virtual-machine:/home/jch# docker pull mysql:5.7
5.7: Pulling from library/mysql
72a69066d2fe: Already exists 
93619dbc5b36: Already exists 
99da31dd6142: Already exists 
626033c43d70: Already exists 
37d5d7efb64e: Already exists 
ac563158d721: Already exists 
d2ba16033dad: Already exists 
0ceb82207cd7: Pull complete 
37f2405cae96: Pull complete 
e2482e017e53: Pull complete 
70deed891d42: Pull complete 
Digest: sha256:f2ad209efe9c67104167fc609cca6973c8422939491c9345270175a300419f94
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
````
````
# 查看本地镜像
root@jch-virtual-machine:/home/jch# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    9c7a54a9a43c   7 months ago    13.3kB
mysql         5.7       c20987f18b13   23 months ago   448MB
mysql         latest    3218b38490ce   23 months ago   516MB
````
* 4、docker rmi 删除镜像
````
docker rmi -f IMAGE ID                        # 删除指定镜像

root@jch-virtual-machine:/home/jch# docker rmi -f 3218b38490ce
Untagged: mysql:latest
Untagged: mysql@sha256:e9027fe4d91c0153429607251656806cc784e914937271037f7738bd5b8e7709
Deleted: sha256:3218b38490cec8d31976a40b92e09d61377359eab878db49f025e5d464367f3b
Deleted: sha256:aa81ca46575069829fe1b3c654d9e8feb43b4373932159fe2cad1ac13524a2f5
Deleted: sha256:0558823b9fbe967ea6d7174999be3cc9250b3423036370dc1a6888168cbd224d
Deleted: sha256:a46013db1d31231a0e1bac7eeda5ad4786dea0b1773927b45f92ea352a6d7ff9
Deleted: sha256:af161a47bb22852e9e3caf39f1dcd590b64bb8fae54315f9c2e7dc35b025e4e3
Deleted: sha256:feff1495e6982a7e91edc59b96ea74fd80e03674d92c7ec8a502b417268822ff
root@jch-virtual-machine:/home/jch# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    9c7a54a9a43c   7 months ago    13.3kB
mysql         5.7       c20987f18b13   23 months ago   448MB
````
````
docker rmi -f IMAGE ID1 IMAGE ID2 IMAGE ID3   # 删除多个镜像
````
````
docker rmi -f $(docker images -aq)            # 删除所有镜像

root@jch-virtual-machine:/home/jch# docker rmi -f $(docker images -aq)
Untagged: hello-world:latest
Untagged: hello-world@sha256:c79d06dfdfd3d3eb04cafd0dc2bacab0992ebc243e083cabe208bac4dd7759e0
Deleted: sha256:9c7a54a9a43cca047013b82af109fe963fde787f63f9e016fdc3384500c2823d
Untagged: mysql:5.7
Untagged: mysql@sha256:f2ad209efe9c67104167fc609cca6973c8422939491c9345270175a300419f94
Deleted: sha256:c20987f18b130f9d144c9828df630417e2a9523148930dc3963e9d0dab302a76
Deleted: sha256:6567396b065ee734fb2dbb80c8923324a778426dfd01969f091f1ab2d52c7989
Deleted: sha256:0910f12649d514b471f1583a16f672ab67e3d29d9833a15dc2df50dd5536e40f
Deleted: sha256:6682af2fb40555c448b84711c7302d0f86fc716bbe9c7dc7dbd739ef9d757150
Deleted: sha256:5c062c3ac20f576d24454e74781511a5f96739f289edaadf2de934d06e910b92
Deleted: sha256:8805862fcb6ef9deb32d4218e9e6377f35fb351a8be7abafdf1da358b2b287ba
Deleted: sha256:872d2f24c4c64a6795e86958fde075a273c35c82815f0a5025cce41edfef50c7
Deleted: sha256:6fdb3143b79e1be7181d32748dd9d4a845056dfe16ee4c827410e0edef5ad3da
Deleted: sha256:b0527c827c82a8f8f37f706fcb86c420819bb7d707a8de7b664b9ca491c96838
Deleted: sha256:75147f61f29796d6528486d8b1f9fb5d122709ea35620f8ffcea0e0ad2ab0cd0
Deleted: sha256:2938c71ddf01643685879bf182b626f0a53b1356138ef73c40496182e84548aa
Deleted: sha256:ad6b69b549193f81b039a1d478bc896f6e460c77c1849a4374ab95f9a3d2cea2
root@jch-virtual-machine:/home/jch# docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
````
## 3、容器命令
* 1、有镜像才可创建容器
````
docker pull centos
````
* 2、新建容器并启动
````
docker run [可选参数] image
 
# 参数说明
--name=“Name”   容器名字    tomcat01    tomcat02    用来区分容器
-d      后台方式运行
-it     使用交互方式运行，进入容器查看内容
-p      指定容器的端口     -p 8080:8080
    -p  ip:主机端口：容器端口
    -p  主机端口：容器端口（常用）
    -p  容器端口
    容器端口
-p      随机指定端口
````
````
# 测试，启动并进入容器
root@jch-virtual-machine:/# docker run -it centos /bin/bash
[root@ffb9eaf76fe1 /]# ls # 查看容器内的centos，基础版本，很多命令是不完善的
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
````
````
# 从容器中退回主机
[root@ffb9eaf76fe1 /]# exit
exit
root@jch-virtual-machine:/# ls
bin    dev   initrd.img      lib64       mnt   root  snap      sys  var
boot   etc   initrd.img.old  lost+found  opt   run   srv       tmp  vmlinuz
cdrom  home  lib             media       proc  sbin  swapfile  usr
````
* 3、列出所有的运行的容器
````
# docker ps 命令
        # 列出当前正在运行的容器
-a      # 列出正在运行的容器包括历史容器
-n=?    # 显示最近创建的容器
-q      # 只显示当前容器的编号

root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@jch-virtual-machine:/# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED         STATUS                       PORTS     NAMES
ffb9eaf76fe1   centos         "/bin/bash"   3 minutes ago   Exited (130) 2 minutes ago             kind_bartik
ae6f178eb9ef   9c7a54a9a43c   "/hello"      2 days ago      Exited (0) 2 days ago                  bold_booth
f866f068cc06   9c7a54a9a43c   "/hello"      2 days ago      Exited (0) 2 days ago                  friendly_raman
f6507b309d6f   9c7a54a9a43c   "/hello"      2 days ago      Exited (0) 2 days ago                  stupefied_jemison
root@jch-virtual-machine:/# docker ps -a -n=1
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                        PORTS     NAMES
ffb9eaf76fe1   centos    "/bin/bash"   15 minutes ago   Exited (130) 14 minutes ago             kind_bartik
root@jch-virtual-machine:/# docker ps -qa
ffb9eaf76fe1
ae6f178eb9ef
f866f068cc06
f6507b309d6f
````
* 4、退出容器
````
exit            # 直接退出容器并关闭
Ctrl + P + Q    # 容器不关闭退出
root@jch-virtual-machine:/# docker run -it centos /bin/bash
[root@5ce250b248e1 /]# root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
5ce250b248e1   centos    "/bin/bash"   54 seconds ago   Up 53 seconds             dreamy_sutherland
````
* 5、删除容器
````
docker rm 容器id                 # 删除指定容器
root@jch-virtual-machine:/# docker ps -a
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS                        PORTS     NAMES
5ce250b248e1   centos         "/bin/bash"   3 minutes ago    Up 3 minutes                            dreamy_sutherland
ffb9eaf76fe1   centos         "/bin/bash"   21 minutes ago   Exited (130) 20 minutes ago             kind_bartik
ae6f178eb9ef   9c7a54a9a43c   "/hello"      2 days ago       Exited (0) 2 days ago                   bold_booth
f866f068cc06   9c7a54a9a43c   "/hello"      2 days ago       Exited (0) 2 days ago                   friendly_raman
f6507b309d6f   9c7a54a9a43c   "/hello"      2 days ago       Exited (0) 2 days ago                   stupefied_jemison
root@jch-virtual-machine:/# docker rm ffb9eaf76fe1
ffb9eaf76fe1

#不能删除正在运行的容器, 如果需要则改成 rm -f
root@jch-virtual-machine:/# docker rm 5ce250b248e1
Error response from daemon: You cannot remove a running container 5ce250b248e159644baf3a68fb617272e3093ccb9f09992e01c86f64411b8d0f. Stop the container before attempting removal or force remove
````
````
docker rm -f $(docker ps -aq)       # 删除所有容器
root@jch-virtual-machine:/# docker ps -aq
5ce250b248e1
ae6f178eb9ef
f866f068cc06
f6507b309d6f
root@jch-virtual-machine:/# docker rm -f $(docker ps -aq)
5ce250b248e1
ae6f178eb9ef
f866f068cc06
f6507b309d6f
root@jch-virtual-machine:/# docker ps -aq
````
````
docker ps -a -q|xargs docker rm -f  # 删除所有的容器

````
* 6、启动和停止容器的操作
````
docker start 容器id           # 启动容器
docker restart 容器id         # 重启容器
docker stop 容器id            # 停止当前正在运行的容器
docker kill 容器id            # 强制停止当前的容器

root@jch-virtual-machine:/# docker run -it centos /bin/bash
[root@84258ba98bcb /]# exit
exit
root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@jch-virtual-machine:/# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES
84258ba98bcb   centos    "/bin/bash"   33 seconds ago   Exited (0) 20 seconds ago             friendly_brown
root@jch-virtual-machine:/# docker start 84258ba98bcb
84258ba98bcb
root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS          PORTS     NAMES
84258ba98bcb   centos    "/bin/bash"   About a minute ago   Up 10 seconds             friendly_brown
root@jch-virtual-machine:/# docker stop 84258ba98bcb
84258ba98bcb
root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
````
## 4、常用的其他命令
* 1、后台启动容器
````
# 命令 docker run -d 镜像名
root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@jch-virtual-machine:/# docker run -d centos
e15cf4679edf7254fa8484dc66f58d805c391bd3d4f6e268b480e0298d62adf8
root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@jch-virtual-machine:/# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES
e15cf4679edf   centos    "/bin/bash"   16 seconds ago   Exited (0) 14 seconds ago             loving_lehmann
84258ba98bcb   centos    "/bin/bash"   27 minutes ago   Exited (0) 26 minutes ago             friendly_brown

# 问题 docker ps, 发现centos停止了
# 常见的坑, docker 容器使用后台运行, 就必须要有一个前台进程,docker发现没有应用,就会自动停止
# 比如docker安装nginx, 容器启动后,发现自己没有提供服务,就会立即停止,就是没有程序了
````
* 2、查看日志
````
root@jch-virtual-machine:/# docker run -d centos /bin/bash
273680e5977fb828d0fcdc0174623bde958876266ca5ebe56771cb777ed00188

root@jch-virtual-machine:/# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS                      PORTS     NAMES
273680e5977f   centos    "/bin/bash"   21 seconds ago   Exited (0) 18 seconds ago             vibrant_leakey

# docker logs -f -t --tail number 容器id
root@jch-virtual-machine:/# docker logs -f -t --tail 10 273680e5977f #没有日志输出
````
````
# 自己编写一段shell脚本
root@jch-virtual-machine:/# docker run -d centos /bin/sh -c "while true;do echo java;sleep 1;done"
f397cd4af25da088cb79c19242ef4df27114b12d6ccd01b1f14923e449c22afb
root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND                   CREATED         STATUS         PORTS     NAMES
f397cd4af25d   centos    "/bin/sh -c 'while t…"   5 seconds ago   Up 4 seconds             crazy_knuth
root@jch-virtual-machine:/# docker logs -f -t --tail 10 f397cd4af25d
2023-12-11T07:47:42.016427118Z java
2023-12-11T07:47:43.019801328Z java
2023-12-11T07:47:44.023397733Z java
2023-12-11T07:47:45.026562254Z java
2023-12-11T07:47:46.029603357Z java
2023-12-11T07:47:47.033131374Z java
2023-12-11T07:47:48.036512643Z java
2023-12-11T07:47:49.040345170Z java
2023-12-11T07:47:50.044498938Z java
2023-12-11T07:47:51.048043825Z java
2023-12-11T07:47:52.052078160Z java
2023-12-11T07:47:53.054080516Z java
2023-12-11T07:47:54.057865386Z java
2023-12-11T07:47:55.061704352Z java

# 显示日志
-f -t                 # 显示日志
--tail number       # 显示日志条数
````
* 3、查看容器中进程信息
````
# 命令 docker top 容器id
root@jch-virtual-machine:/# docker top f397cd4af25d 
UID                 PID                 PPID                C                   STIME               TTY 
root                26408               26375               0                   15:44               ?  
root                27060               26408               0                   15:54               ?  
````
* 4、查看镜像的元数据
````
# docker inspect 容器id
root@jch-virtual-machine:/# docker inspect f397cd4af25d
````
* 6、进入当前正在运行的容器
````
# 我们通常容器使用后台方式运行的, 需要进入容器，修改一些配置

# 方式一
# docker exec -it 容器id /bin/bash
root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND                   CREATED          STATUS          PORTS     NAMES
f397cd4af25d   centos    "/bin/sh -c 'while t…"   20 minutes ago   Up 20 minutes             crazy_knuth
root@jch-virtual-machine:/# docker exec -it f397cd4af25d /bin/bash
[root@f397cd4af25d /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
[root@f397cd4af25d /]# ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 07:44 ?        00:00:00 /bin/sh -c while true;do echo java;sleep 1;done
root       1260      0  0 08:05 pts/0    00:00:00 /bin/bash
root       1359      1  0 08:07 ?        00:00:00 /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
root       1360   1260  0 08:07 pts/0    00:00:00 ps -ef

# 方式二
# docker attach 容器id
root@jch-virtual-machine:/# docker attach f397cd4af25d

# docker exec       # 进入容器后开启一个新的终端，可以在里面操作
# docker attach     # 进入容器正在执行的终端，不会启动新的进程
````
* 7、从容器中拷贝文件到主机
````
# docker cp 容器id：容器内路径    目的地主机路径

# 启动容器
root@jch-virtual-machine:/# docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
centos       latest    5d0da3dc9764   2 years ago   231MB
root@jch-virtual-machine:/# docker run -it centos /bin/bash
[root@35d734d870f3 /]# root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
35d734d870f3   centos    "/bin/bash"   38 seconds ago   Up 36 seconds             xenodochial_kapitsa

# 进入docker容器内部创建文件
root@jch-virtual-machine:/# docker attach 35d734d870f3
[root@35d734d870f3 /]# cd /home
[root@35d734d870f3 home]# ls
[root@35d734d870f3 home]# touch test.java
[root@35d734d870f3 home]# exit  
exit

从容器内拷贝文件到主机上
root@jch-virtual-machine:/# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
root@jch-virtual-machine:/# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED         STATUS                      PORTS     NAMES
35d734d870f3   centos    "/bin/bash"   6 minutes ago   Exited (0) 16 seconds ago             xenodochial_kapitsa
root@jch-virtual-machine:/# docker cp 35d734d870f3:/home/test.java /home
Successfully copied 1.54kB to /home
root@jch-virtual-machine:/# cd /home
root@jch-virtual-machine:/home# ls
jch  test.java
````