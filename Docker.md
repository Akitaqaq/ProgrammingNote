## Docker 基本概念

略

##  安装Docker

### 安装docker

```sh
#1.卸载旧版
 yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
#2.需要的安装包
 yum install -y yum-utils
#3.设置存储库
 yum-config-manager \ 
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo #官方地址
------------------------------------------------------------------------------------
 yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo #阿里云镜像加速地址
    
#4.更新yum软件包
 yum makecache fast
# 安装docker docker-ce-cli ce 社区版本 ee 企业版本
 yum install docker-ce docker-ce-cli containerd.io
#5.启动docker
     service docker start
#systemctl start docker
#6.使用docker version 是否安装成功
```

 ![](https://gitee.com/amiaosixsix/md-img/raw/master/img/image-20210319155926911.png)

```shell
#7.验证docker是否安装正确
docker run hello-world 
#查看镜像
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
hello-world   latest    d1165f221234   13 days ago   13.3kB

```

卸载docker

```shell
#卸载依赖
yum remove docker-ce docker-ce-cli containerd.io
#删除依赖
rm -rf /var/lib/docker
rm -rf /var/lib/containerd
```

### 阿里云镜像加速

1、找到镜像加速位置

![image-20210319162730993](https://gitee.com/amiaosixsix/md-img/raw/master/img/image-20210319162730993.png)

![image-20210319162617959](https://gitee.com/amiaosixsix/md-img/raw/master/img/image-20210319162617959.png)

2、配置镜像加速

```shell
sudo mkdir -p /etc/docker

sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://1jhlnnzf.mirror.aliyuncs.com"]
}
EOF

sudo systemctl daemon-reload

sudo systemctl restart docker
```

## Docker 常用命令

	#### 帮助命令

```shell
docker -命令 --help #帮助命令
docker info 	   #显示docker系统信息，包含镜像和容器的数量
```



### 镜像命令

**docker images**  #查看所有本地主机上的镜像

``` shell
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
hello-world   latest    d1165f221234   13 days ago   13.3kB
#释义
REPOSITORY	 镜像的仓库源
TAG          镜像的标签
IMAGE ID 	 镜像的id 
CREATED      镜像的创建时间
SIZE		镜像的大小
```

**docker  search**  #搜索镜像

```shell
[root@localhost ~]# docker search mysql
'^HNAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   10629     [OK]       
mariadb                           MariaDB Server is a high performing open sou…   3987      [OK]       
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   779                  [OK]
#可选项
--filter=STARS=3000 #过滤数据
```

**docker  pull** 下载镜像

```shell
#下载镜像 docker pull 镜像名[:tag]
[root@localhost ~]# docker pull mysql
Using default tag: latest #如果不写tag，默认是latest
latest: Pulling from library/mysql 
a076a628af6f: Pull complete #分层下载
f6c208f3f991: Pull complete 
88a9455a9165: Pull complete 
406c9b8427c6: Pull complete 
7c88599c0b25: Pull complete 
25b5c6debdaf: Pull complete 
43a5816f1617: Pull complete 
1a8c919e89bf: Pull complete 
9f3cf4bd1a07: Pull complete 
80539cea118d: Pull complete 
201b3cad54ce: Pull complete 
944ba37e1c06: Pull complete 
Digest: sha256:feada149cb8ff54eade1336da7c1d080c4a1c7ed82b5e320efb5beebed85ae8c #签名
Status: Downloaded newer image for mysql:latest 
docker.io/library/mysql:latest #真实地址

#等价
docker pull mysql
docker pull docker.io/library/mysql:latest

#指定版本下载
[root@localhost ~]# docker pull mysql:5.7
5.7: Pulling from library/mysql
a076a628af6f: Already exists 
f6c208f3f991: Already exists 
88a9455a9165: Already exists 
406c9b8427c6: Already exists 
7c88599c0b25: Already exists 
25b5c6debdaf: Already exists 
43a5816f1617: Already exists 
1831ac1245f4: Pull complete 
37677b8c1f79: Pull complete 
27e4ac3b0f6e: Pull complete 
7227baa8c445: Pull complete 
Digest: sha256:b3d1eff023f698cd433695c9506171f0d08a8f92a0c8063c1a4d9db9a55808df
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7


```

**docker rmi** #删除镜像

```shell
docker rmi -f 镜像id #删除指定镜像id
docker rmi -f 镜像id 镜像id 镜像id #删除多个镜像
docker rmi -f $(docker images -aq) #删除全部镜像
```

### 容器命令

**说明：有镜像才可以创建容器，下载centos测试**

```shell
docker pull centos
```

**新建容器并启动**

```shell
#容器启动
docker run [参数名称] centos
#可选参数
--name="name" 指定容器名称
-d			 后台交互运行
-it			 使用交互方式运行，进入容器
-p			 指定容器的端口 -p 8080:8080
	#几种指定方式
	-p ip:主机端口：容器端口
	-p 主机端口：容器端口
	-p 容器端口
-p 			 随机指定端口
```

**测试并进入容器内**

```shell
#进入容器
[root@localhost ~]# docker run -it centos
[root@430ddf75938c /]# ls #查看内部centos
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var
dev  home  lib64  media       opt  root  sbin  sys  usr
#退出容器
[root@430ddf75938c /]# exit
```

**查看当前运行运行中的容器**

```shell
#docker ps 命令
#可选项
-a 		#列出当前和历史运行过的容器
-n=?	#显示最近创建的容器
-q
[root@localhost /]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
73fec05f69ce   centos    "/bin/bash"   12 seconds ago   Up 12 seconds             jovial_driscoll
```

**退出容器**

```shell
#直接退出后台不运行
exit
#容器退出后台运行
Ctrl + P + Q
```

**删除容器**

```shell
docker rm 容器id 				 #删除指定的容器，不能删除正在运行的容器，强制删除 rm -f
docker rm -f $(docker ps -aq) 	#删除所用容器
docker ps -a -q|xargs docker rm #删除所用容器
```

**启动和停止容器的操作**

```shell
docker start 容器id   #启动容器
docker restart 容器id #重启容器
docker stop 容器id    #停止当前运行的容器
docker kill 容器id    #强制停止当前容器
```

### 其它常用命令

**后台启动容器**

```shell
#命令 docker run -itd 镜像名！
[root@localhost /]# docker run -itd 300e315adb2f
#自启动
docker update cef8c24d12fe --restart=always
```

**查看日志**

```shell
docker logs -f -t --tail 容器
```

**查看容器内的进程id**

```shell
docker top 容器id
[root@localhost /]# docker top e7b0b6265a6e
UID     PID      PPID     C     STIME    TTY     TIME       CMD
root    6068     6045     0     09:59    pts/0   00:00:00   /bin/bash
```

**查看容器的元数据**

```shell
[root@localhost /]# docker inspect e7b0b6265a6e
[
    {
        "Id": "e7b0b6265a6e8dbf869318f4fae656a7cd6b2614dc2e4c6021ffe0d7bc1a16f5",
        "Created": "2021-03-22T01:59:02.093430314Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 6068,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-03-22T01:59:02.50749276Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },....
]
```

**进入当前正在运行的容器**

```shell
#方式一 命令 docker exec -it 容器id /bin/bash
[root@localhost /]# docker exec -it e7b0b6265a6e /bin/bash
[root@e7b0b6265a6e /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

#方式二 docker attach 容器id
[root@localhost /]# docker attach e7b0b6265a6e
[root@e7b0b6265a6e /]# ls
bin  etc   lib	  lost+found  mnt  proc  run   srv  tmp  var

#两种方式区别
#docker exec	#进入容器开启一个新终端
#docker attach  #进入容器不会开启新终端
```

**从容器拷贝文件到主机**

待写

### docker命令图

![](https://gitee.com/amiaosixsix/md-img/raw/master/img/docker1.jpg)

### docker安装nginx

```shell
#1.搜索镜像，使用官网或者命令搜索;官网：https://hub.docker.com/search?q=nginx&type=image
#2.拉取镜像
[root@localhost /]# docker pull nginx
#3.运行测试
[root@localhost /]# docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    f6d0b4767a6c   2 months ago   133MB
centos       latest    300e315adb2f   3 months ago   209MB
#--name  给容器命名 
#-d	     后台运行
#-p		宿主机端口:容器内容端口
[root@localhost /]# docker run --name nginx01 -d -p 8080:80 nginx
4e26825f5924669a26e7d1531b759f2993c0da09258d1ab366d0036cd833f983
[root@localhost /]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                  NAMES
4e26825f5924   nginx     "/docker-entrypoint.…"   9 seconds ago   Up 7 seconds   0.0.0.0:8080->80/tcp   nginx01
[root@localhost /]# curl localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
#进入容器
[root@localhost /]# docker exec -it 4e26825f5924 /bin/bash
root@4e26825f5924:/# cd etc/nginx/
root@4e26825f5924:/etc/nginx# ls
conf.d	fastcgi_params	koi-utf  koi-win  mime.types  modules  nginx.conf  scgi_params	uwsgi_params  win-utf
root@4e26825f5924:/etc/nginx# 

```

### docker安装tomcat

```shell
#拉取镜像
[root@localhost /]# docker pull tomcat:8.5.64
#启动容器
docker run --name tomcat01 -d -p tomcat
#进入容器
[root@localhost ~]# docker exec -it 2111a7942e27 /bin/bash
root@2111a7942e27:/usr/local/tomcat# ls
BUILDING.txt	 LICENSE  README.md	 RUNNING.txt  conf  logs	    temp     webapps.dist
CONTRIBUTING.md  NOTICE   RELEASE-NOTES  bin	      lib   native-jni-lib  webapps  work
#默认文件在webapps.dist中，webapps默认空
root@2111a7942e27:/usr/local/tomcat# cd webapps
root@2111a7942e27:/usr/local/tomcat/webapps# ls
#赋值默认文件到webapps
root@2111a7942e27:/usr/local/tomcat# cp -r webapps.dist/* webapps
root@2111a7942e27:/usr/local/tomcat# cd webapps
root@2111a7942e27:/usr/local/tomcat/webapps# ls
ROOT  docs  examples  host-manager  manager

```

## docker镜像原理

待写

#### commit镜像

```shell
#命令
docker commit -m="提交的描述信息" -a="作者" 容器id 目标镜像名:[TAG]
```

测试

```shell
#1.启动一个tomcat
#2.默认官网提供镜像webapps为空，复制一些基本文件到wepapps
#3.将操作过的容器commid提交为一个镜像！
```

![image-20210323095809169](https://gitee.com/amiaosixsix/md-img/raw/master/img/image-20210323095809169.png)

## docker容器卷

简介，待写

> 直接使用命令挂载

```shell
docker run it -v 主机目录:容器内目录
#测试
[root@localhost ~]# docker run -it -v /home/ceshi:/home centos /bin/bash

```

![image-20210323135447968](https://gitee.com/amiaosixsix/md-img/raw/master/img/image-20210323135447968.png)

![image-20210323135913776](https://gitee.com/amiaosixsix/md-img/raw/master/img/image-20210323135913776.png)

### 安装mysql测试

```shell
#安装mysql
[root@localhost /]# docker pull mysql:5.7
#安装启动mysql。需要配置数据库密码，否则无法启动
-d 后台运行
-p 端口映射
-v 卷挂载
-e 环境配置
[root@localhost home]# docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:5.7
#第三方工具连接测试，端口配置为映射的3310，连接成功
```

容器删除，测试数据是否持久化

```shell
[root@localhost data]# docker rm -f mysql01 #删除容器
mysql01
[root@localhost data]# ls  #查看数据，还存在
auto.cnf    client-cert.pem  ibdata1      ibtmp1              private_key.pem  server-key.pem
ca-key.pem  client-key.pem   ib_logfile0  mysql               public_key.pem   sys
ca.pem      ib_buffer_pool   ib_logfile1  performance_schema  server-cert.pem  test

```

### 具名和匿名挂载

```shell
#匿名挂载：只指定容器内的，不指定容器外的 注意：P是大写 
[root@localhost data]# docker run -d -P --name nginx02 -v /ect/nginx/ nginx
#查看所有volume的信息
[root@localhost data]# docker volume ls
DRIVER    VOLUME NAME
local     c843cb8854a3224db57733ec8ba56fee12aace6d93b3a360df5894f0a07d2608  #匿名挂载格式
#具名挂载
[root@localhost data]# docker run -d -P --name nginx03 -v jumingNginx:/ect/nginx/ nginx
[root@localhost data]# docker volume ls
DRIVER    VOLUME NAME
local     c843cb8854a3224db57733ec8ba56fee12aace6d93b3a360df5894f0a07d2608  
local     jumingNginx #具名挂载格式
#查看具体的挂载info
[root@localhost data]# docker volume inspect jumingNginx
[
    {
        "CreatedAt": "2021-03-23T17:00:18+08:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/jumingNginx/_data", #挂载路径
        "Name": "jumingNginx",
        "Options": null,
        "Scope": "local"
    }
]
```

所有docker容器的卷，没有指定的情况下默认在 ==/var/lib/docker/volumes/xxx/_data== 

```shell
#区分匿名和具名挂载
-v 容器内路径 #匿名挂载
-v 卷名:容器内路径 #具名挂载
-v /宿主机路径:容器内路径 #指定路径挂载
```