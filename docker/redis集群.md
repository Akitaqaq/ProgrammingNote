## redis

### 1.安装docker

```shell
#官方的一键安装方式
curl -sSL https://get.daocloud.io/docker | sh
#启动
sudo systemctl start docker
```

### 2.安装redis

#### 2.1拉取镜像

````shell
docker search redis
docker pull redis
#创建网段
docker network create myredis 
````



#### 2.2 脚本创建redis挂载配置文件

````shell
for port in $(seq 7000 7005); 
do 
mkdir -p /home/redis/node-${port}/conf
touch /home/redis/node-${port}/conf/redis.conf
cat  << EOF > /home/redis/node-${port}/conf/redis.conf
port ${port}
requirepass 1234
bind 0.0.0.0
protected-mode no
daemonize no
appendonly yes
notify-keyspace-events Ex
cluster-enabled yes 
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 10.73.236.226
cluster-announce-port ${port}
cluster-announce-bus-port 1${port}
EOF
done

````

### 3.启动容器

````shell
for port in $(seq 7000 7005); \
do \
   docker run -it -d -p ${port}:${port} -p 1${port}:1${port} \
  --privileged=true -v /home/redis/node-${port}/conf/redis.conf:/usr/local/etc/redis/redis.conf \
  --privileged=true -v /home/redis/node-${port}/data:/data \
  --restart always --name redis-${port} --net myredis \
  --sysctl net.core.somaxconn=1024 redis redis-server /usr/local/etc/redis/redis.conf
done

````

#### 3.1进入容器

````shell
docker exec -it redis-7000 /bin/bash
````

#### 3.2 创建集群

````shell
redis-cli  -a 1234 --cluster create 10.73.236.226:7000 10.73.236.226:7001 10.73.236.226:7002 10.73.236.226:7003 10.73.236.226:7004 10.73.236.226:7005   --cluster-replicas 1
````

#### 3.5 进入redis

```shell
redis-cli -h 10.73.236.226 -p 7001 -c -a 1234
#203
redis-cli -h 10.73.236.203 -p 6381 -c

CONFIG GET notify-keyspace-events
CONFIG SET notify-keyspace-events Ex
```



## docker

容器操作：

````shell
#一条命令实现停用并删除容器
docker stop $(docker ps -q) & docker rm $(docker ps -aq)
#停止所有容器
docker stop $(docker ps -q)
#启动所有容器
docker start $(docker ps -a | awk '{ print $1}' | tail -n +2)
````

## 单机伪集群

```shell
sed -i 's/7291/7292/g' 7292/redis.conf
sed -i 's/7291/7293/g' 7293/redis.conf
sed -i 's/7291/7294/g' 7294/redis.conf
sed -i 's/7291/7295/g' 7295/redis.conf
sed -i 's/7291/7296/g' 7296/redis.conf

#集群启动
/data/smp/redis-6.0.9/src/redis-server /data/smp/redis-cluster/7291/redis.conf
/data/smp/redis-6.0.9/src/redis-server /data/smp/redis-cluster/7292/redis.conf
/data/smp/redis-6.0.9/src/redis-server /data/smp/redis-cluster/7293/redis.conf
/data/smp/redis-6.0.9/src/redis-server /data/smp/redis-cluster/7294/redis.conf
/data/smp/redis-6.0.9/src/redis-server /data/smp/redis-cluster/7295/redis.conf
/data/smp/redis-6.0.9/src/redis-server /data/smp/redis-cluster/7296/redis.conf


redis-cli   --cluster create 10.73.236.226:7291 10.73.236.226:7292 10.73.236.226:7293 10.73.236.226:7294 10.73.236.226:7295 10.73.236.226:7296   --cluster-replicas 1

redis-cli -p 7291

sed -i 's/notify-keyspace-events Ex/notify-keyspace-events AKE/g' /data/smp/redis-cluster/*/redis.conf
```

