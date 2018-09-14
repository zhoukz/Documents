# zookeeper+hadooop+hbase+spark 安装步骤

## 一、数据规划
|HOST				  |HADOOP									|ZOOKEEPER			|	HBASE 		   |spark  |
|-----------  |-----------------------|---------------|--------------|------ |
|**gonggao09**|NameNode								|QuorumPeerMain	|HMaster 	     |Master |
|**gonggao09**|JournalNode						|	--            |HRegionServer |Worker |
|**gonggao09**|DFSZKFailoverController|	--	          | --           | --    |
|**gonggao09**|ResourceManager			  | --            | --           | --    |
|**gonggao09**|DataNode			          | --            | --           | --    |
|**gonggao09**|NodeManager			      | --            | --           |--     |
|**gonggao10**|NameNode								|QuorumPeerMain	|HMaster     	 |Worker |
|**gonggao10**|JournalNode						|	--            | HRegionServer| --    |
|**gonggao10**|DFSZKFailoverController|	--	          | --           |--     |
|**gonggao10**|DataNode			          | --            | --           |--     |
|**gonggao10**|NodeManager			      | --            | --           |--     |
|**gonggao11**|DataNode								|QuorumPeerMain	|HRegionServer |Worker |
|**gonggao11**|NodeManager			      | --            | --           |--     |
## 二、zookeeper 安装

* 2.1  用root新建登录用户和密码：
```
usrgrop hadoop
useradd -g hadoop -d /uniiof/users/zkpusr01 -m zkpusr01
passd  zkpusr01 stdin <<< zkpusr01
```
* 2.2 上传zookeeper和jdk安装包到/opt/freeware并解压
```
cd /opt/freeware
tar –zxvf zookeeper-3.4.6.tar.gz
tar –zxvf jdk1.7.0_51.tgz
```
* 2.3 配置hosts
```
vi /etc/host
192.168.170.30 gonggao09
192.168.170.31 gonggao10
192.168.170.32 gonggao11
```
* 2.4 登录zkpusr01用户并创建目录
```
su - zkpusr01
mkdir -p zookeeper/bin zookeeper/logs zookeeper/conf zookeeper/zk-data
```
* 2.5 配置环境变量
```
vi $HOME/.bash_profile
export JAVA_HOME=/opt/freeware/jdk1.7.0_51
export PATH=$JAVA_HOME/bin:$PATH
export ZOOBINDIR=/opt/freeware/zookeeper
export ZOOKEEPER_PREFIX=/opt/freeware/zookeeper
export ZOOCFGDIR=/uniiof/users/zkpusr01/zookeeper/conf
export ZOO_LOG_DIR=/uniiof/users/zkpusr01/zookeeper/logs
export PATH=$PATH:$ZOOBINDIR/bin
```
* 2.6 复制执行文件和脚本
```
cp $ZOOKEEPER_PREFIX/bin/* ~/zookeeper/bin
```
* 2.7 复制配置文件
```
cp $ZOOKEEPER_PREFIX/conf/* ~/zookeeper/conf
cp ~/zookeeper/conf/zoo_sample.cfg ~/zookeeper/conf/zoo.cfg
```
* 2.8 修改配置文件:
```
在$HOME/zookeeper/conf 下，添加和更改zoo.cfg 配置文件
vi zoo.cfg
tickTime=40000
dataDir=/uniiof/users/zkpusr01/zookeeper/zk-data
clientPort=29181
server.1=gonggao09:29888:39888
server.2=gonggao10:29888:39888
server.3=gonggao11:29888:39888
```
* 2.9 将zookpeer目录拷贝到其他2个节点
```
scp -r zkpusr01@gonggao10:/uniiof/users/zkpusr01
scp -r zkpusr01@gonggao11:/uniiof/users/zkpusr01
```
* 2.10 修改所有节点上的myid
```
cd /uniiof/users/zkpusr01/zookeeper/zk-data
vi myid
1
注: myid 根据上面zoo.cfg配置而设置
例:
server.1=gonggao09 myid=1
server.2=gonggao10 myid=2
Server.3=gonggao11 myid=3
```
* 2.11 启动zookeeper:
```
cd /uniiof/users/zkpusr01/zookeeper/bin
[zkpusr01@gonggao09 bin]$ ./bin/zkServer.sh start
[zkpusr01@gonggao09 bin]$ jps
125021 Jps
93303 QuorumPeerMain
说明服务起来了
```
* 2.12 查看状态:
```
[zkpusr01@gonggao09 zookeeper]$ ./bin/zkServer.sh status
JMX enabled by default
Using config: /uniiof/users/zkpusr01/zookeeper/conf/zoo.cfg
Mode: follower
```
* 2.13 登录其他节点启动zookeeper

## 三 hadoop 安装
* 3.1 新建hadoop用户
```
useradd -g hadoop -d /uniiof/users/hdpusr01  -m hdpusr01
passwd hdpusr01 --stdin <<< hdpusr01
```
* 3.2 上传安装包到/opt/freeware目录并解压
```
tar -zxvf  hadoop-2.5.1.tar.gz
```
* 3.3 登录hdpusr01配置节点互信
```
[hdpusr01@gonggao09~]$ ssh-keygen -t rsa
[hdpusr01@gonggao09 ~]$ ssh-copy-id hdpusr01@gonggao09
[hdpusr01@gonggao09 ~]$ ssh-copy-id hdpusr01@gonggao10
[hdpusr01@gonggao09 ~]$ ssh-copy-id hdpusr01@gonggao11
```
* 3.4 登录hdpusr01用户并新建目录
```
mkdir hadoop
cd hadoop
mkdir -p yarn/nm-local-dir yarn/userlogs pid logs etc tmp bin libexec
cd /data
mkdir -p dfs1/ dfs1/dn dfs1/nn
```
* 3.5 配置环境变量
```
cd $HOME
vi .bash_profile
export JAVA_HOME=/opt/freeware/jdk1.7.0_51
export PATH=.:$JAVA_HOME/bin:$PATH
export HADOOP_CONF_DIR=$HOME/hadoop/etc
export HADOOP_PID_DIR=$HOME/hadoop/pid
export HADOOP_LOG_DIR=$HOME/hadoop/logs
export HADOOP_HOME=/opt/freeware/hadoop-2.5.1
export HADOOP_LIBEXEC_DIR=$HOME/hadoop/libexec
export HADOOP_PREFIX=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export HADOOP_LIB=${HADOOP_HOME}/lib
export HADOOP_LIBRARY_PATH=${HADOOP_HOME}/lib/native
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:${HADOOP_HOME}/lib/native
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_YARN_USER=hdpusr01
export YARN_CONF_DIR=${HADOOP_CONF_DIR}
export YARN_HOME=${HADOOP_HOME}
export HADOOP_YARN_HOME=${HADOOP_HOME}
export YARN_LOG_DIR=${HADOOP_LOG_DIR}
export YARN_PID_DIR=${HADOOP_CONF_DIR}/../yarn
export HADOOP_COMMON_LIB_NATIVE_DIR=${HADOOP_PREFIX}/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_PREFIX/lib"
```
* 3.6 拷贝启动文件和配置文件
```
cd ~/hadoop/bin
cp $HADOOP_HOME/sbin/* ./
cd ~/hadoop/etc
cp $HADOOP_HOME/etc/hadoop/* .
cd ~/hadoop/libexec
cp $HADOOP_HOME/libexec/* .
```
* 3.7 修改配置文件
```
cd $HOME/hadoop/ect
```

* 3.7.1 配置master和slaves文件
```
vi master
gonggao09
gonggao10
vi slaves
gonggao09
gonggao10
gonggao11
```
* 3.7.2 配置core-site.xml文件，vi core-site.xml
```
<configuration>
<property>
<name>fs.defaultFS</name>
<value>hdfs://hdpcls1/</value>
<description>The name of the default file system.A URI whose scheme and authority determine the FileSystem implementation.The uri's scheme determines the config property (fs.SCHEME.impl) naming the FileSystem implementation class.The uri's authority is used to determine the host, port, etc. for a filesystem.</description>
</property>
<property>
<name>dfs.ha.fencing.methods</name>
<value>sshfence</value>
<description>
List of fencing methods to use for service fencing. May contain builtin methods (eg shell and sshfence) or user-defined method.
</description>
</property>
<property>
<name>dfs.ha.fencing.ssh.private-key-files</name>
<value>/uniiof/users/hdpusr01/.ssh/id_rsa</value>
<description>
The SSH private key files to use with the builtin sshfence fencer.
</description>
</property>
<property>
<name>dfs.ha.fencing.ssh.connect-timeout</name>
<value>30000</value>
<description>
SSH connection timeout, in milliseconds, to use with the builtin sshfence fencer.
</description>
</property>
<property>
<name>ha.zookeeper.quorum</name>
<value>gonggao09:29181,gonggao10:29181,gonggao11:29181</value>
<description>
A list of ZooKeeper server addresses, separated by commas, that are to be used by the ZKFailoverController in automatic failover.
</description>
</property>
<property>
<name>hadoop.tmp.dir</name>
<value>/uniiof/users/hdpusr01/hadoop/tmp</value>
 <description>A base for other temporary directories.</description>
</property>
<property>
<name>io.native.lib.available</name>
<value>true</value>
<description>Should native hadoop libraries, if present, be used.</description>
</property>
<property>
<name>fs.trash.interval</name>
<value>0</value>
<description>Number of minutes after which the checkpoint
gets deleted.If zero, the trash feature is disabled.This option may be configured both on the server and the client. If trash is disabled server side then the client side configuration is checked. If trash is enabled on the server side then the value configured on the server is used and the client configuration value is ignored.
</description>
</property>
</configuration>
```
* 3.7.3 配置hdfs-site.xml文件vi hdfs-site.xml
```
<configuration>
<property>
<name>dfs.nameservices</name>
<value>hdpcls1</value>
<description>
Comma-separated list of nameservices.
</description>
</property>
<property>
<name>dfs.ha.namenodes.hdpcls1</name>
<value>nn1,nn2</value>
<description>
The prefix for a given nameservice, contains a comma-separated list of namenodes for a given nameservice (eg EXAMPLENAMESERVICE).
</description>
</property>
<property>
<name>dfs.namenode.rpc-address.hdpcls1.nn1</name>
<value>gonggao09:8920</value>
</property>
<property>
<name>dfs.namenode.rpc-address.hdpcls1.nn2</name>
<value>gonggao10:8920</value>
</property>
<property>
<name>dfs.namenode.http-address.hdpcls1.nn1</name>
<value>gonggao09:59070</value>
</property>
<property>
<name>dfs.namenode.http-address.hdpcls1.nn2</name>
<value>gonggao10:59070</value>
</property>
<property>
<name>dfs.journalnode.edits.dir</name>
<value>/data/dfs1/jn</value>
</property>
<property>
<name>dfs.journalnode.rpc-address</name>
<value>0.0.0.0:8485</value>
<description>
The JournalNode RPC server address and port.
</description>
</property>
<property>
<name>dfs.journalnode.http-address</name>
<value>0.0.0.0:8480</value>
<description>
The address and port the JournalNode web UI listens on. If the port is 0 then the server will start on a free port.
</description>
</property>
<property>
<name>dfs.client.failover.proxy.provider.hdpcls1</name>
<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
</property>
<property>
<name>dfs.ha.automatic-failover.enabled</name>
<value>true</value>
<description>
Whether automatic failover is enabled. See the HDFS High Availability documentation for details on automatic HA
configuration.
</description>
</property>
<property>
<name>dfs.namenode.name.dir</name>
<value>/data/dfs1/nn</value>
<description>Determines where on the local filesystem the DFS name node should store the name table(fsimage).If this is a comma-delimited list of directories then the name table is replicated in all of the directories, for redundancy. </description>
</property>
<property>
<name>dfs.datanode.data.dir</name>
<value>/data/dfs1/dn</value>
<description>Determines where on the local filesystem an DFS data node should store its blocks.If this is a comma-delimited list of directories, then data will be stored in all named directories, typically on different devices.Directories that do not exist are ignored.
</description>
</property>
<property>
<name>dfs.replication</name>
<value>3</value>
<description>Default block replication.
The actual number of replications can be specified when the file is created. The default is used if replication is not specified in create time.
</description>
</property>
<property>
<name>dfs.permissions.enabled</name>
<value>true</value>
<description>
If "true", enable permission checking in HDFS.
If "false", permission checking is turned off,but all other behavior is unchanged.
Switching from one parameter value to the other does not change the mode,owner or group of files or directories.
</description>
</property>
<property>
<name>dfs.permissions.superusergroup</name>
<value>hadoop</value>
<description>The name of the group of super-users.</description>
</property>
<property>
<name>dfs.datanode.hdfs-blocks-metadata.enabled</name>
<value>true</value>
<description>
Boolean which enables backend datanode-side support for the experimental DistributedFileSystem#getFileVBlockStorageLocations API.
</description>
</property>
<property>
 <name>dfs.datanode.failed.volumes.tolerated</name>
<value>0</value>
<description>The number of volumes that are allowed to fail before a datanode stops offering service. By default any volume failure will cause a datanode to shutdown.
</description>
</property>
<property>
<name>dfs.namenode.secondary.http-address</name>
<value>gonggao09:59090</value>
<description>
The secondary namenode http server address and port.If the port is 0 then the server will start on a free port.
</description>
</property>
<property>
<name>dfs.https.enable</name>
<value>false</value>
<description>
Decide if HTTPS(SSL) is supported on HDFS
</description>
</property>
<property>
<name>dfs.namenode.https-address</name>
<value>0.0.0.0:59470</value>
<description>
The namenode secure http server address and port.
</description>
</property>
<property>
<name>dfs.datanode.address</name>
<value>0.0.0.0:59010</value>
<description>
The datanode server address and port for data transfer.If the port is 0 then the server will start on a free port.
</description>
</property>
<property>
<name>dfs.datanode.http.address</name>
<value>0.0.0.0:59075</value>
<description>
The datanode http server address and port.If the port is 0 then the server will start on a free port.
</description>
</property>
<property>
<name>dfs.datanode.https.address</name>
<value>0.0.0.0:59475</value>
<description>
The datanode secure http server address and port.
</description>
</property>
<property>
<name>dfs.datanode.ipc.address</name>
<value>0.0.0.0:59020</value>
<description>
The datanode ipc server address and port.If the port is 0 then the server will start on a free port.
</description>
</property>
<property>
<name>dfs.namenode.backup.address</name>
<value>0.0.0.0:59100</value>
<description>
The backup node server address and port.If the port is 0 then the server will start on a free port.
</description>
</property>
<property>
<name>dfs.namenode.backup.http-address</name>
<value>0.0.0.0:59105</value>
<description>
The backup node http server address and port.If the port is 0 then the server will start on a free port.
</description>
</property>
<property>
<name>dfs.datanode.balance.bandwidthPerSec</name>
<value>104857600</value>
<description>
Specifies the maximum amount of bandwidth that each datanode can utilize for the balancing purpose in term of the number of bytes per second.
</description>
</property>
<property>
<name>dfs.datanode.du.reserved</name>
<value>107374182400</value>
<description>Reserved space in bytes per volume. Always leave this much space free for non dfs use.
</description>
</property>
<property>
<name>dfs.datanode.max.xcievers</name>
<value>8192</value>
</property>
</configuration>
```
* 3.7.4 配置yarn-site.xml文件,vi yarn-site.xml
```
<configuration>
<property>
<description>The address of the applications manager interface in the RM.</description>
<name>yarn.resourcemanager.address</name>
<value>gonggao09:8932</value>
</property>
<property>
<description>The address of the scheduler interface.</description>
<name>yarn.resourcemanager.scheduler.address</name>
<value>gonggao09:8930</value>
</property>
<property>
<description>The address of the RM web application.</description>
<name>yarn.resourcemanager.webapp.address</name>
<value>gonggao09:8988</value>
</property>
<property>
<name>yarn.resourcemanager.resource-tracker.address</name>
<value>gonggao09:8931</value>
</property>
<property>
<description>The address of the RM admin interface.</description>
<name>yarn.resourcemanager.admin.address</name>
<value>gonggao09:8933</value>
</property>
<!-- Node Manager Configs -->
<property>
<description>The address of the container manager in the NM.</description>
<name>yarn.nodemanager.address</name>
<value>0.0.0.0:8941</value>
</property>
<property>
<description>
List of directories to store localized files in. An application's localized file directory will be found in: ${yarn.nodemanager.local-dirs}/usercache/${user}/appcache/application_${appid}. Individual containers' work directories, called container_${contid}, will be subdirectories of this.
</description>
<name>yarn.nodemanager.local-dirs</name>
<!-- <value>${hadoop.tmp.dir}/nm-local-dir</value> -->
<value>/uniiof/users/hdpusr01/hadoop/yarn/nm-local-dir</value> ->
</property>
<property>
<description>Address where the localizer IPC is.</description>
<name>yarn.nodemanager.localizer.address</name>
<value>0.0.0.0:8940</value>
</property>
<property>
<description>
Where to store container logs. An application's localized log directory will be found in ${yarn.nodemanager.log-dirs}/application_${appid}. Individual containers' log directories will be below this, in directories named container_{$contid}. Each container directory will contain the files stderr, stdin, and syslog generated by that container.
</description>
<name>yarn.nodemanager.log-dirs</name>
<!-- <value>${yarn.log.dir}/userlogs</value> -->
<value>/uniiof/users/hdpusr01/hadoop/yarn/userlogs</value>
</property>
<property>
<description>Where to aggregate logs to.</description>
<name>yarn.nodemanager.remote-app-log-dir</name>
<!-- <value>/tmp/logs</value> -->
<value>/tmp/logs/hadoop-yarn/apps</value>
</property>
<property>
<description>NM Webapp address.</description>
<name>yarn.nodemanager.webapp.address</name>
<value>0.0.0.0:8942</value>
</property>
<!-- WebAppProxy Configuration-->
<property>
<description>
CLASSPATH for YARN applications. A comma-separated list of CLASSPATH entries</description>
<name>yarn.application.classpath</name>
<value>$HADOOP_CONF_DIR,$HADOOP_COMMON_HOME/share/hadoop/common/*,$HADOOP_COMMON_HOME/share/hadoop/common/lib/*,$HADOOP_HDFS_HOME/share/hadoop/hdfs/*,$HADOOP_HDFS_HOME/share/hadoop/hdfs/lib/*,$YARN_HOME/share/hadoop/yarn/*,$YARN_HOME/share/hadoop/yarn/lib/*,$YARN_HOME/share/hadoop/mapreduce/*,$YARN_HOME/share/hadoop/mapreduce/lib/*</value>
</property>
<property>
<name>yarn.nodemanager.aux-services</name>
<value>mapreduce_shuffle</value>
</property>
</configuration>
```
* 3.7.5 配置mapred-site.xml文件,vi mapred-site.xml
```
<configuration>
<property>
<name>mapreduce.jobtracker.address</name>
<value>gonggao09:8921</value>
<description>
The host and port that the MapReduce job tracker runs at. If "local", then jobs are run in-process as a single map and reduce task.
</description>
</property>
<property>
<name>mapreduce.jobtracker.http.address</name>
<value>gonggao09:59030</value>
<description>
The job tracker http server address and port the server will listen on.If the port is 0 then the server will start on a free port.
</description>
</property>
<property>
<name>mapreduce.cluster.local.dir</name>
<value>/uniiof/users/hdpusr01/hadoop/mapred/local</value>
description>The local directory where MapReduce stores intermediate data files. May be a comma-separated list of directories on different devices in order to spread disk i/o. Directories that do not exist are ignored.
</description>
</property>
<property>
<name>mapreduce.jobtracker.system.dir</name>
<!-- <value>${hadoop.tmp.dir}/mapred/system</value> -->
<value>/tmp/mapred/system</value>
<description>
The directory where MapReduce stores control files.
</description>
</property>
<property>
<name>mapreduce.jobtracker.staging.root.dir</name>
<!-- <value>${hadoop.tmp.dir}/mapred/staging</value> -->
<value>/tmp/mapred/staging</value>
<description>
The root of the staging area for users' job files In practice, this should be the directory where users' home directories are located (usually /user)
</description>
</property>
<property>
<name>mapreduce.cluster.temp.dir</name>
<!-- <value>${hadoop.tmp.dir}/mapred/temp</value> -->
<value>/tmp/mapred/temp</value>
<description>
A shared directory for temporary files.
</description>
</property>
<property>
<name>mapreduce.job.maps</name>
<value>2</value>
<description>
The default number of map tasks per job.Ignored when mapreduce.jobtracker.address is "local".
</description>
</property>
<property>
<name>mapreduce.job.reduces</name>
<value>1</value>
<description>
The default number of reduce tasks per job. Typically set to 99% of the cluster's reduce capacity, so that if a node fails the reduces can still be executed in a single wave. Ignored when mapreduce.jobtracker.address is "local".
</description>
</property>
<property>
<name>mapreduce.tasktracker.http.address</name>
<value>0.0.0.0:59060</value>
<description>
The task tracker http server address and port.If the port is 0 then the server will start on a free port.
</description>
</property>
<property>
<name>mapreduce.shuffle.port</name>
<value>8980</value>
<description>
Default port that the ShuffleHandler will run on. ShuffleHandler is a service run at the NodeManager to facilitate transfers of intermediate Map outputs to requesting Reducers.
</description>
</property>
<!-- MR YARN Application properties -->
<property>
<name>mapreduce.framework.name</name>
<value>yarn</value>
<description>
The runtime framework for executing MapReduce jobs.Can be one of local, classic or yarn.
</description>
</property>
<property>
<name>yarn.app.mapreduce.am.staging-dir</name>
<value>/tmp/hadoop-yarn/staging</value>
<description>
The staging dir used while submitting jobs.
</description>
</property>
<property>
<name>yarn.app.mapreduce.am.command-opts</name>
<value>-Xmx1024m</value>
<description>
Java opts for the MR App Master processes. The following symbol, if present, will be interpolated: @taskid@ is replaced by current TaskID. Any other occurrences of '@' will go unchanged. For example, to enable verbose gc logging to a file named for the taskid in /tmp and to set the heap maximum to be a gigabyte, pass a 'value' of: -Xmx1024m -verbose:gc -Xloggc:/tmp/@taskid@.gc Usage of -Djava.library.path can cause programs to no longer function if hadoop native libraries are used. These values should instead be set as part of LD_LIBRARY_PATH in the map / reduce JVM env using the mapreduce.map.env and mapreduce.reduce.env config settings.
</description>
</property>
<property>
<name>yarn.app.mapreduce.am.resource.mb</name>
<value>1536</value>
<description>
The amount of memory the MR AppMaster needs.</description>
</property>
<property>
<name>mapreduce.application.classpath</name>
<value>
$HADOOP_CONF_DIR,
$HADOOP_COMMON_HOME/share/hadoop/common/*,
$HADOOP_COMMON_HOME/share/hadoop/common/lib/*,
$HADOOP_HDFS_HOME/share/hadoop/hdfs/*,
$HADOOP_HDFS_HOME/share/hadoop/hdfs/lib/*,
$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/*,
$HADOOP_MAPRED_HOME/share/hadoop/mapreduce/lib/*,
$HADOOP_YARN_HOME/share/hadoop/yarn/*,
$HADOOP_YARN_HOME/share/hadoop/yarn/lib/*
</value>
</property>
</configuration>
```
* 3.7.6 配置hadoop-env.sh文件，vi hadoop-env.sh
```
export JAVA_HOME=/opt/freeware/jdk1.7.0_51
export PATH=.:$JAVA_HOME/bin:$PATH
export HADOOP_CONF_DIR=$HOME/hadoop/etc
export HADOOP_PID_DIR=$HOME/hadoop/pid
export HADOOP_LOG_DIR=$HOME/hadoop/logs
export HADOOP_HOME=/opt/freeware/hadoop-2.5.1
export HADOOP_LIBEXEC_DIR=$HOME/hadoop/libexec
export HADOOP_PREFIX=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export HADOOP_LIB=${HADOOP_HOME}/lib
export HADOOP_LIBRARY_PATH=${HADOOP_HOME}/lib/native
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:${HADOOP_HOME}/lib/native
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```
* 3.7.7 配置yarn-env.sh文件，vi yarn-env.sh
```
export JAVA_HOME=/opt/freeware/jdk1.7.0_51
export PATH=.:$JAVA_HOME/bin:$PATH
export HADOOP_CONF_DIR=$HOME/hadoop/etc
export HADOOP_PID_DIR=$HOME/hadoop/pid
export HADOOP_LOG_DIR=$HOME/hadoop/logs
export HADOOP_HOME=/opt/freeware/hadoop-2.5.1
export HADOOP_LIBEXEC_DIR=$HOME/hadoop/libexec
export HADOOP_PREFIX=${HADOOP_HOME}
export HADOOP_COMMON_HOME=${HADOOP_HOME}
export HADOOP_HDFS_HOME=${HADOOP_HOME}
export HADOOP_LIB=${HADOOP_HOME}/lib
export HADOOP_LIBRARY_PATH=${HADOOP_HOME}/lib/native
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:${HADOOP_HOME}/lib/native
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=${HADOOP_HOME}
export HADOOP_YARN_USER=hdpusr01
export YARN_CONF_DIR=${HADOOP_CONF_DIR}
export YARN_HOME=${HADOOP_HOME}
export HADOOP_YARN_HOME=${HADOOP_HOME}
export YARN_LOG_DIR=${HADOOP_LOG_DIR}
export YARN_PID_DIR=${HADOOP_CONF_DIR}/../yarn
```
* 3.8 讲hadoop目录同步到其他所有节点的机器上
```
cd $HOME
scp -r hadoop hdpusr01@gonggao10:/uniiof/users/hdpusr01
scp -r hadoop hdpusr01@gonggao11:/uniiof/users/hdpusr01
```
* 3.9 启动hadoop集群
* 3.9.1 初始化HA 状态到zookeeper
```
hdfs zkfc -formatZK
```
* 3.9.2 初始化部署和启动HA
```
在主、备节点（gonggao09、gonggao10）的hdpusr01 下操作如下：
cd ~/hadoop/bin
./hadoop-daemon.sh start journalnode
```
* 3.9.3 初始化和启动NameNode
```
在gonggao09执行：
hdfs namenode -fromat
cd $home/hadoop/bin
./hadoop-daemon.sh start namenode
```
* 3.9.4 要把节点gonggao09上的元数据复制到节点gonggao10上去
```
scp -r /data/dfs1/nn hdpusr01@gonggao10:/data/dfs1/
```
* 3.9.5 在gonggao10启动NameNode
```
cd $HOME/hadoop/bin
./hadoop-daemon.sh start namenode
```
* 3.9.6 启动所有数据节点和mapreduce
```
./start-dfs.sh
./start-yarn.sh
```
* 3.10 查看启动进程
* 3.10.1在gonggao09执行：jps
```
NameNode
JournalNode
DFSZKFailoverController
ResourceManager
DataNode
NodeManager
```
* 3.10.2在gonggao10执行：jps
```
NameNode
JournalNode
DFSZKFailoverController
DataNode
NodeManager
```
* 3.10.3在gonggao11执行：jps
```
DataNode
NodeManager
```
* 3.10 查看dfs和集群状态信息
```
http://192.168.170.30:59070/dfshealth.html
http://192.168.170.30:8988/cluster
```
## 四、hbase安装
* 4.1 上传安装安装包到/opt/freeware，并解压
```
tar -zxvf hbase-0.98.4-hadoop2-bin.tar.gz
```
* 4.2 登录hdpusr01用户，并创建hbase目录
```
mkdir -p hbase/bin hbase/conf hbase/logs hbase/pid hbase/tmp
```
* 4.3 配置环境变量,vi $HOME/.bash_profile
```
export JAVA_HOME=/opt/freeware/jdk1.7.0_21
export PATH=.:$JAVA_HOME/bin:$PATH
export ZOOBINDIR=/uniiof/users/zkpusr01/zookeeper
export ZOOKEEPER_PREFIX=/uniiof/users/zkpusr01/zookeeper
export ZOOCFGDIR=/uniiof/users/zkpusr01/zookeeper/conf
export ZOO_LOG_DIR=/uniiof/users/zkpusr01/zookeeper/logs
export PATH=$PATH:$ZOOBINDIR/bin
export HADOOP_HOME=/opt/freeware/hadoop-2.5.1
export HBASE_HOME=/opt/freeware/hbase-0.98.4-hadoop2-bin
export HBASE_CONF_DIR=/uniiof/users/hbsusr01/hbase/conf
export HBASE_PID_DIR=/uniiof/users/hbsusr01/hbase/pid
export HBASE_LOG_DIR=/uniiof/users/hbsusr01/hbase/logs
export PATH=$PATH:$HBASE_HOME/bin
```
* 4.4 复制启动和配置文件
```
cp -rp /opt/freeware/hbase-0.98.4-hadoop2-bin/*  ~/hbase/bin
cp /opt/freeware/hbase-0.98.4-hadoop2-bin/conf/*  ~/hbase/conf
```
* 4.5 修改配置文件，cd $HOME/hbase/conf
* 4.5.1 配置regionservers，vi regionservers
```
gonggao09
gonggao10
gonggao11
```
* 4.5.2 配置hbase-site.xml，vi hbase-site.xml
```
<configuration>
<property>
<name>hbase.rootdir</name>
<value>hdfs://hdpcls1/hbase</value>
</property>
<property>
<name>hbase.master.port</name>
<value>60900</value>
<description>
The port the HBase Master should bind to.
</description>
</property>
<property>
<name>hbase.cluster.distributed</name>
<value>true</value>
<description>
The mode the cluster will be in. Possible values are false for standalone mode and true for distributed mode. If false, startup will run all HBase and ZooKeeper daemons togetherin the one JVM.
</description>
</property>
<property>
<name>hbase.tmp.dir</name>
<!-- <value>/tmp/hbase-${user.name}</value> -->
<value>/uniiof/users/hbsusr01/hbase/tmp</value>
<description>
Temporary directory on the local filesystem. Change this setting to point to a location more permanent than '/tmp' (The '/tmp' directory is often cleared on machine restart).
</description>
</property>
<property>
<name>hbase.master.info.port</name>
<value>60910</value>
<description>
The port for the HBase Master web UI.Set to -1 if you do not want a UI instance run.
</description>
</property>
<property>
<name>hbase.regionserver.port</name>
<value>60920</value>
<description>
The port the HBase RegionServer binds to.
</description>
</property>
<property>
<name>hbase.regionserver.info.port</name>
<value>60930</value>
<description>
The port for the HBase RegionServer web UISet to -1 if you do not want the RegionServer UI to run.
</description>
</property>
<!--
  The following three properties are used together to create the list ofhost:peer_port:leader_port quorum servers for ZooKeeper.
-->
<property>
<name>hbase.zookeeper.quorum</name>
<value>gonggao09,gonggao10,gonggao11</value>
<description>
Comma separated list of servers in the ZooKeeper Quorum.For example, "host1.mydomain.com,host2.mydomain.com,host3.mydomain.com".By default this is set to localhost for local and pseudo-distributed modes of operation. For a fully-distributed setup, this should be set to a full list of ZooKeeper quorum servers. If HBASE_MANAGES_ZK is set in hbase-env.sh this is the list of servers which we will start/stop ZooKeeper on.
</description>
</property>
<property>
<name>hbase.zookeeper.peerport</name>
<value>29888</value>
<description>
Port used by ZooKeeper peers to talk to each other.See http://hadoop.apache.org/zookeeper/docs/r3.1.1/zookeeperStarted.html#sc_RunningReplicatedZoo Keeper for more information.
</description>
</property>
<property>
<name>hbase.zookeeper.leaderport</name>
<value>39888</value>
<description>
Port used by ZooKeeper for leader election.See http://hadoop.apache.org/zookeeper/docs/r3.1.1/zookeeperStarted.html#sc_RunningReplicated ZooKeeper for more information.
</description>
</property>
<!-- End of properties used to generate ZooKeeper host:port quorum list. -->
<property>
<name>hbase.zookeeper.property.clientPort</name>
<value>29181</value>
<description>Property from ZooKeeper's config zoo.cfg.The port at which the clients will connect.
</description>
</property>
<!-- End of properties that are directly mapped from ZooKeeper's zoo.cfg -->
<property>
<name>hbase.rest.port</name>
<value>8980</value>
<description>The port for the HBase REST server.</description>
</property>
</configuration>
```
* 4.5.3 配置hbase-env.sh，vi hbase-env.sh
```
export JAVA_HOME=/opt/freeware/jdk1.7.0_51         
export PATH=.:$JAVA_HOME/bin:$PATH                     
export HBASE_HOME=/opt/freeware/hbase-0.98.4-hadoop2-bin
export HBASE_CONF_DIR=/uniiof/users/hbsusr01/hbase/conf
export HBASE_PID_DIR=/uniiof/users/hbsusr01/hbase/pid
export HBASE_LOG_DIR=/uniiof/users/hbsusr01/hbase/logs
export HBASE_MANAGES_ZK=false
```
* 4.5.4 配置core-site.xml和hdfs-site.xml
```
ln –s /uniiof/users/hdpusr01/hadoop/etc/core-site.xml core-site.xml
ln –s /uniiof/users/hdpusr01/hadoop/etc/hdfs-site.xml hdfs-site.xml
```
* 4.6 拷贝hbase目录文件到其他主机
```
scp -r hbase/ hdpusr01@gonggao10:/uniiof/users/hdpusr01
scp -r hbase/ hdpusr01@gonggao11:/uniiof/users/hdpusr01
```
* 4.7 启动hbase集群
* 4.7.1 在gonggao09主机启动Hmaster和HRegionServer节点
```
cd $HOME/hbase/bin
./start-hbase.sh
```
* 4.7.2 在gonggao10主机启动备用Hmaster
```
cd $HOME/hbase/bin
./ hbase-daemon.sh start master --backup
```
* 4.8 查看启动进程
* 4.8.1在gonggao09执行：jps
```
HMaster
HRegionServer
```
* 4.8.2在gonggao10执行：jps
```
HMaster
HRegionServer
```
* 3.8.3在gonggao11执行：jps
```
HRegionServer
```
* 4.9 查看hbase集群状态：
```
http://192.168.170.30:60910/master-status
```
## 五、spark安装
* 5.1 上传安装包到hdpusr01用户HOME目录，并解压
```
tar -zxvf spark-1.5.1-bin-hadoop2.4.tar.gz
```
* 5.2 启动spark
* 5.2.1 启动spark启动主节点（在gonggao09主机上执行）：
```
cd $HOME/spark-1.5.1-bin-hadoop2.4/sbin
start-master.sh -h 192.168.170.30
```
* 5.2.1 启动spark启动备节点（在gonggao09、10和11三台主机上执行）：
```
cd $HOME/spark-1.5.1-bin-hadoop2.4/sbin
start-slave.sh spark://192.168.170.30:7077 -m 8g
```
* 5.3 查看启动进程
* 5.3.1在gonggao09执行：jps
```
Master
Worker
```
* 5.3.2在gonggao10执行：jps
```
Worker
```
* 5.3.3在gonggao11执行：jps
```
Worker
```
* 5.4 查看spark集群状态：
```
http://192.168.170.30:8080/
```
