# redis+kafka+strom 安装步骤
## 一、redis安装
* 1.1 上传安装包到/opt/freeware 目录，并解压
```
tar -zxvf redis-2.8.13.tar.gz
```
* 1.2 登录到zkpusr01用户，并创建redis目录：
```
su - zkpusr01
mkdir redis
cd redis
mkdir bin conf logs redis-db
```
* 1.3 配置环境变量，vi $HOME/.bash_profile
```
export REDIS_HOME=/opt/freeware/redis-2.8.13
export PATH=${REDIS_HOME}/bin:${PATH}
```
* 1.4 配置 redis.conf
```
cd $HOME/redis/conf
vi redis.conf
logfile /uniiof/users/zkpusr01/redis/logs/redis.log
port 16379
dir  /uniiof/users/zkpusr01/redis/logs/redis-db/
pidfile  /uniiof/users/zkpusr01/redis/redis-db/redis.pid
```
* 1.5 配置sentine.conf
```
cd $HOME/redis/conf
vi sentine.conf
port 26379
sentinel monitor mymaster 192.168.170.30 16379 1    
sentinel down-after-milliseconds mymaster 5000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
```
* 1.6 配置主用reids 启动脚本，vi redis_start.sh(在gonggao09执行)
```
ps -ef|grep redis-server|grep 16379|grep -v grep|awk '{print $2}'|xargs kill -9;
redis-server /uniiof/users/zkpusr01/redis/conf/redis.conf &
```
* 1.7 配置备用redis 启动脚本,vi redis_start.sh（在gonggao10执行）
```
ps -ef|grep redis-server|grep 16379|grep -v grep|awk '{print $2}'|xargs kill -9;
redis-server $HOME/redis/conf/redis.conf slaveof 192.168.170.30 16379 --port 16379 &
```
* 1.8 配置sentine启动脚本，vi start_sentine.sh（在gonggao09、10和11执行）
```
#!/bin/bash
ps -ef|grep redis-server|grep 26379|grep -v grep|awk '{print $2}'|xargs kill -9;
redis-server $HOME/redis/conf/sentinel.conf --sentinel &> $HOME/redis/logs/sentinel.log &
```
* 1.9 授予启动脚本执行权限并启动redis和sentine
```
cd $HOME/redis/bin
chmod 755 start_redis.sh
chmod 755 start_sentinel.sh
./start_redis.sh
./start_sentinel.sh
```
* 1.10 查看redis状态
* 1.10.1 查看主用redis状态
```
redis-cli -h 192.168.170.30 -p 16379 INFO
```
* 1.10.2 查看备用用redis状态
```
redis-cli -h 192.168.170.31 -p 16379 INFO
```
* 1.10.3 查看sentinel状态
```
redis-cli -h 192.168.170.30 -p 26379 INFO
redis-cli -h 192.168.170.31 -p 26379 INFO
redis-cli -h 192.168.170.32 -p 26379 INFO
```
* 1.11 查看redis进程
```
ps -ef |grep redis
```
## 二、kakfa 安装

* 2.1 创建用户
```
user add -d /uniiof/users/kfkusr01 -m kfkusr01
passwd kfkusr01 --stdin <<< kfkusr01
```
* 2.2 上次安装包到用户kfkusr的HOME目录并解压
```
tar -xvf kafka_2.10-0.8.1.1.tgz
```
* 2.3 修改配置文件，cd $HONE/kafka_2.10-0.8.1.1/config
* 2.3.1 vi consumer.properties
```
zookeeper.connect=gonggao09:29181,gonggao10:29181,gonggao11:29181
#timeout in ms for zookeeper session
zookeeper.session.timeout.ms=50000
#rebalance timeout settings : rebalance.max.retries * rebalance.backoff.ms > zookeeper.session.timeout.ms
rebalance.backoff.ms=10000
rebalance.max.retries=10
```
* 2.3.2 vi producer.properties
```
metadata.broker.list=gonggao09:19092,gonggao10:19092,gonggao11:19092
```
* 2.3.3  vi server.properties
```
#(用IP地址的最后一位标示，其他主机用31和32，方便查看问题)
broker.id=30
#(修改端口号)
port=19092
#(修改主机,其他主机根据主机的IP修改)
host.name=192.168.170.30
advertised.host.name=192.168.170.30
#log数据目录data 没有要自己新建
log.dirs=/uniiof/users/kfkusr01/kafka_2.10-0.8.1.1/data
#zookeeper.connect=localhost:2181(修改zk集群)
zookeeper.connect=gonggao09:29181,gonggao10:29182,gonggao11:29181/kafka
```
* 2.3.4 vi zookeeper.properties
```
#(zookeeper目录要新建)
dataDir=/uniiof/users/kfkusr01/kafka_2.10-0.8.1.1/zookeeper
#zk端口号
clientPort=29181
```
* 2.4 配置启动脚本
```
#sbin目录要创建
cd $HOME/kafka_2.10-0.8.1.1/sbin
vi start_kafka.sh
nohup /uniiof/users/kfkusr01/kafka_2.10-0.8.1.1/bin/kafka-server-start.sh /uniiof/users/kfkusr01/kafka_2.10-0.8.1.1/config/server.properties  > /uniiof/users/kfkusr01/kafka_2.10-0.8.1.1/logs/kafka.log 2>&1 &
```
* 2.5 授予启动脚本执行权限并启动kafka
```
cd $HOME/kafka_2.10-0.8.1.1/sbin
chmod 755 start_kafka.sh
./ start_kafka.sh
```
* 2.6 查看进程，jps
```
Kafka
```
## 三、storm 安装

* 3.1 上传安装包到kfkusr01用户HOME目录，并解压
```
tar -xvf storm-0.9.5.tar.gz
```
* 3.2 修改配置文件
```
cd $HOME/storm-0.9.5/conf
vi storm.yaml
storm.local.dir: /uniiof/users/kfkusr01/storm-0.9.5/stmdata
storm.zookeeper.servers:
    - "gonggao09"
    - "gonggao10"
    - "gonggao11"
storm.zookeeper.port: 29181
storm.zookeeper.root: wastorm
nimbus.host: 192.168.170.30
nimbus.thrift.port: 19192
ui.port: 19292
supervisor.slots.ports:
    - 26700
    - 26701
    - 26702
    - 26703
    - 26704
    - 26705
    - 26706
    - 26707
```
* 3.3 storm启动
* 3.3.1 启动主节点（在gonggao09主机上执行）
```
cd $HOME/storm-0.9.5/bin
./storm nimbus &
./storm ui &
```
* 3.3.2 启动备节点（在gonggao09、10和11主机上执行）
```
cd $HOME/storm-0.9.5/bin
./storm supervisor &
```
* 3.4 查看进程
* 3.4.1 在gonggao09主机上执行，jps
```
supervisor
nimbus
core
```
* 3.4.2 在gonggao10主机上执行，jps
```
supervisor
```
* 3.4.3 在gonggao11主机上执行，jps
```
supervisor
```
* 3.5 查看storm UI管理界面
```
http://192.168.170.30:19292/index.html
```
