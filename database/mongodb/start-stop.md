# 启动与停止

## :pencil2: 1、启动 <a href="#1-qi-dong" id="1-qi-dong"></a>

### :pen\_fountain: 1.1、启动命令 <a href="#11-qi-dong-ming-ling" id="11-qi-dong-ming-ling"></a>

```
# 前台启动
mongod --dbpathD:\workspace\mongodb\data
# 后台启动
mongod -dbpath=/usr/local/mongodb/data --fork --port 27017 --logpath=/usr/local/mongodb/log/work.log --logappend --auth
```

### :pen\_fountain: 1.2、注册为服务 <a href="#12-zhu-ce-wei-fu-wu" id="12-zhu-ce-wei-fu-wu"></a>

```
mongod --logpath "D:\workspace\mongodb\log\mongodb.log" --logappend--dbpath "D:\workspace\mongodb\data" --port 27017 --serviceName "mongodbService"--serviceDisplayName "mongodbService" --installmongo
```

### :pen\_fountain: 1.3、`Mongod`命令参数 <a href="#13mongod-ming-ling-can-shu" id="13mongod-ming-ling-can-shu"></a>

#### 1.基本配置

```
--quiet           # 安静输出
--port arg        # 指定服务端口号，默认端口27017
--bind_ip arg     # 绑定服务IP，若绑定127.0.0.1，则只能本机访问，不指定默认本地所有IP
--logpath arg     # 指定MongoDB日志文件，注意是指定文件不是目录
--logappend       # 使用追加的方式写日志
--pidfilepath arg # PID File 的完整路径，如果没有设置，则没有PID文件
--keyFile arg     # 集群的私钥的完整路径，只对于Replica Set 架构有效
--unixSocketPrefix arg  # UNIX域套接字替代目录,(默认为 /tmp)
--fork            # 以守护进程的方式运行MongoDB，创建服务器进程
--auth            # 启用验证
--cpu             # 定期显示CPU的CPU利用率和iowait
--dbpath arg      # 指定数据库路径
--diaglog arg     # diaglog选项 0=off 1=W 2=R 3=both 7=W+some reads
--directoryperdb  # 设置每个数据库将被保存在一个单独的目录
--journal         # 启用日志选项，MongoDB的数据操作将会写入到journal文件夹的文件里
--journalOptions arg    # 启用日志诊断选项
--ipv6            # 启用IPv6选项
--jsonp           # 允许JSONP形式通过HTTP访问（有安全影响）
--maxConns arg    # 最大同时连接数 默认2000
--noauth          # 不启用验证
--nohttpinterface   # 关闭http接口，默认关闭27018端口访问
--noprealloc      # 禁用数据文件预分配(往往影响性能)
--noscripting     # 禁用脚本引擎
--notablescan     # 不允许表扫描
--nounixsocket    # 禁用Unix套接字监听
--nssize arg (=16)  # 设置信数据库.ns文件大小(MB)
--objcheck        # 在收到客户数据,检查的有效性
--profile arg     # 档案参数 0=off 1=slow, 2=all
--quota           # 限制每个数据库的文件数，设置默认为8
--quotaFiles arg  # number of files allower per db, requires 
--quota--rest     # 开启简单的rest API
--repair          # 修复所有数据库run repair on all dbs
--repairpath arg  # 修复库生成的文件的目录,默认为目录名称dbpath
--slowms arg (=100) # value of slow for profile and console log
--smallfiles      # 使用较小的默认文件
--syncdelay arg (=60)   # 数据写入磁盘的时间秒数(0=never,不推荐)
--sysinfo         # 打印一些诊断系统信息
--upgrade         # 如果需要升级数据库
```

#### 2.`Replicaton` 参数

```
--fastsync      # 从一个dbpath里启用从库复制服务，该dbpath的数据库是主库的快照，可用于快速启用同步
--autoresync    # 如果从库与主库同步数据差得多，自动重新同步
--oplogSize arg # 设置oplog的大小(MB)
```

#### 3.主/从参数

```
--master            # 主库模式
--slave             # 从库模式
--source arg        # 从库 端口号
--only arg          # 指定单一的数据库复制
--slavedelay arg    # 设置从库同步主库的延迟时间
```

#### 4.Replica set(副本集)选项：

```
--replSet arg       # 设置副本集名称
```

#### 5.`Sharding`(分片)选项

```
--configsvr         # 声明这是一个集群的config服务,默认端口27019，默认目录/data/configdb
--shardsvr          # 声明这是一个集群的分片,默认端口27018
--noMoveParanoia    # 关闭偏执为moveChunk数据保存
```

### :pen\_fountain: 1.4、启动方式 <a href="#14-qi-dong-fang-shi" id="14-qi-dong-fang-shi"></a>

#### 1.4.1、基于命令行方式启动`mongodb` <a href="#141-ji-yu-ming-ling-hang-fang-shi-qi-dong-mongodb" id="141-ji-yu-ming-ling-hang-fang-shi-qi-dong-mongodb"></a>

```bash
# mongod --dbpath=/data/mongodata/rs1 --logpath=/data/mongodata/rs1/rs1.log &

缺省端口为
[root@node3 rs1]# netstat -nltp|grep mongod
tcp        0      0 0.0.0.0:27017       0.0.0.0:*       LISTEN      5062/mongod 
```

#### 1.4.2、基于配置文件的命令行启动 <a href="#142-ji-yu-pei-zhi-wen-jian-de-ming-ling-hang-qi-dong" id="142-ji-yu-pei-zhi-wen-jian-de-ming-ling-hang-qi-dong"></a>

```bash
vi /var/lib/mongodb/conf/rs2.conf

port = 27000
dbpath = /data/mongodata/rs2
logpath = /data/mongodata/rs2/rs2.log
smallfiles = true
fork = true
pidfilepath = /var/run/mongo.pid

# mongod --config /var/lib/mongodb/conf/rs2.conf &

### Author : Leshami
### Blog   : http://blog.csdn.net/leshami

# netstat -nltp|grep 27000
tcp        0      0 0.0.0.0:27000       0.0.0.0:*       LISTEN      5356/mongod    
```

#### 1.4.3、以守护进程方式启动`mongodb` <a href="#143-yi-shou-hu-jin-cheng-fang-shi-qi-dong-mongodb" id="143-yi-shou-hu-jin-cheng-fang-shi-qi-dong-mongodb"></a>

```bash
# mongod --dbpath=/data/mongodata/rs3 --logpath=/data/mongodata/rs1/rs3.log --fork --port 28000

# netstat -nltp|grep mongod
tcp        0      0 0.0.0.0:28000           0.0.0.0:*       LISTEN      5465/mongod         
tcp        0      0 0.0.0.0:27017           0.0.0.0:*       LISTEN      5435/mongod         
tcp        0      0 0.0.0.0:27000           0.0.0.0:*       LISTEN      5448/mongod
```

#### 1.4.4、使用系统服务的方式启动`mogodb` <a href="#144-shi-yong-xi-tong-fu-wu-de-fang-shi-qi-dong-mogodb" id="144-shi-yong-xi-tong-fu-wu-de-fang-shi-qi-dong-mogodb"></a>

```bash
# 启动脚本

# vi /etc/init.d/mongod

#!/bin/sh  
# chkconfig: 2345 93 18  

#MogoDB home directory  
MONGODB_HOME=/var/lib/mongodb

#mongodb command  
MONGODB_BIN=$MONGODB_HOME/bin/mongod

#mongodb config file
MONGODB_CONF=$MONGODB_HOME/conf/mongodb.conf

#mongodb PID
MONGODB_PID=/var/run/mongo.pid

#set open file limit
SYSTEM_MAXFD=65535

MONGODB_NAME="mongodb"
. /etc/rc.d/init.d/functions

if [ ! -f $MONGODB_BIN ]
then
     echo "$MONGODB_NAME startup: $MONGODB_BIN not exists! "
     exit
fi

start(){
     ulimit -HSn $SYSTEM_MAXFD     
     $MONGODB_BIN --config="$MONGODB_CONF"  --fork ##added @20160901     
     ret=$?     
     if [ $ret -eq 0 ]; then
         action $"Starting $MONGODB_NAME: " /bin/true     
     else
         action $"Starting $MONGODB_NAME: " /bin/false     
     fi
}

stop(){
     PID=$(ps aux |grep "$MONGODB_NAME" |grep "$MONGODB_CONF" |grep -v grep |wc -l)
     if [[ $PID -eq 0  ]];then
         action $"Stopping $MONGODB_NAME: " /bin/false        
         exit        
     fi        
     kill -HUP `cat $MONGODB_PID`        
     ret=$?        
     if [ $ret -eq 0 ]; then
         action $"Stopping $MONGODB_NAME: " /bin/true                
         rm -f $MONGODB_PID        
     else                   
         action $"Stopping $MONGODB_NAME: " /bin/false        
     fi
}

restart() {
     stop        
     sleep 2        
     start
}

case "$1" in        
     start)    
          start                
          ;;        
     stop)    
          stop                
          ;;        
     status)        
          status $prog                
          ;;        
     restart)                
          restart                
          ;;        
     *)                
          echo $"Usage: $0 {start|stop|status|restart}"
esac

# chmod u+x /etc/init.d/mongod

# service mongod start
about to fork child process, waiting until server is ready for connections.
forked process: 5543
child process started successfully, parent exiting
Starting mongodb:                                          [  OK  ]
```

## :pencil2: 2、停止 <a href="#2-ting-zhi" id="2-ting-zhi"></a>

### :pen\_fountain: 2.1、停止方式 <a href="#21-ting-zhi-fang-shi" id="21-ting-zhi-fang-shi"></a>

#### 2.1.1、向`mongod`进程发送信号 <a href="#211-xiang-mongod-jin-cheng-fa-song-xin-hao" id="211-xiang-mongod-jin-cheng-fa-song-xin-hao"></a>

```bash
###SIGINT信号

# ps -ef|grep mongod|grep rs1
root       5435   4914  1 19:13 pts/2    00:00:14 mongod --dbpath=/data/mongodata/rs1 --logpath=/data/mongodata/rs1/rs1.log

# kill -2 5435

2016-08-30T17:02:00.528+0800 I CONTROL[signalProcessingThread] got signal 2(Interrupt), will terminate after current cmd ends
2016-08-30T17:02:00.530+0800 I REPL     [signalProcessingThread] Stopping replication applier threads
2016-08-30T17:02:00.554+0800 I STORAGE  [conn1253] got request after shutdown()
2016-08-30T17:02:00.774+0800 I CONTROL  [signalProcessingThread] now exiting
2016-08-30T17:02:00.774+0800 I NETWORK  [signalProcessingThread] shutdown: going to close listening sockets...
2016-08-30T17:02:00.774+0800 I NETWORK  [signalProcessingThread] closing listening socket: 6
2016-08-30T17:02:00.775+0800 I NETWORK  [signalProcessingThread] closing listening socket: 7
2016-08-30T17:02:00.775+0800 I NETWORK  [signalProcessingThread] removing socket file: /tmp/mongodb-27017.sock
2016-08-30T17:02:00.775+0800 I NETWORK  [signalProcessingThread] shutdown: going to flush diaglog...
2016-08-30T17:02:00.775+0800 I NETWORK  [signalProcessingThread] shutdown: going to close sockets...
2016-08-30T17:02:00.775+0800 I STORAGE  [signalProcessingThread] shutdown: waiting for fs preallocator...
2016-08-30T17:02:00.775+0800 I STORAGE  [signalProcessingThread] shutdown: final commit...
2016-08-30T17:02:00.775+0800 I JOURNAL  [signalProcessingThread] journalCleanup...
2016-08-30T17:02:00.775+0800 I JOURNAL  [signalProcessingThread] removeJournalFiles
2016-08-30T17:02:00.777+0800 I NETWORK  [conn1254] end connection 192.168.1.247:58349 (0 connections now open)
2016-08-30T17:02:00.779+0800 I JOURNAL  [signalProcessingThread] Terminating durability thread ...
2016-08-30T17:02:00.881+0800 I JOURNAL  [journal writer] Journal writer thread stopped
2016-08-30T17:02:00.882+0800 I JOURNAL  [durability] Durability thread stopped
2016-08-30T17:02:00.882+0800 I STORAGE  [signalProcessingThread] shutdown: closing all files...
2016-08-30T17:02:00.884+0800 I STORAGE  [signalProcessingThread] closeAllFiles() finished
2016-08-30T17:02:00.884+0800 I STORAGE  [signalProcessingThread] shutdown: removing fs lock...
2016-08-30T17:02:00.885+0800 I CONTROL  [signalProcessingThread] dbexit:  rc: 0

###SIGTERM信号
# ps -ef|grep mongod|grep rs3

# ps -ef|grep mongod|grep rs3
root  5465 1 1 19:14 ? 00:00:13 mongod --dbpath=/data/mongodata/rs3 --logpath=/data/mongodata/rs1/rs3.log --fork --port 28000

# kill -4 5465

信号     产生方式 
sigint  通过ctrl+c将会对当进程发送此信号 
sigterm kill命令不加参数就是发送这个信号  

对进程的影响  
sigint 信号被当前进程树接收到，也就是说，不仅当前进程会收到信号，它的子进程也会收到  
sigterm只有当前进程收到信号，子进程不会收到。如果当前进程被kill了，那么它的子进程的父进程将会是init，也就是pid为1的进程

上述信号在发出后：        
不再接受新的连接请求        
等待现有的连接处理完毕        
关闭所有打开的连接        
将内存的数据写出到磁盘        
安全停止
```

#### 2.1.2、使用系统服务脚本方式停止`mongod` <a href="#212-shi-yong-xi-tong-fu-wu-jiao-ben-fang-shi-ting-zhi-mongod" id="212-shi-yong-xi-tong-fu-wu-jiao-ben-fang-shi-ting-zhi-mongod"></a>

```bash
# ps -ef|grep mongod
root   5675  1  3 19:33 ?   00:00:00 /var/lib/mongodb/bin/mongod --config=/var/lib/mongodb/conf/rs2.conf
root       5689   4950  0 19:33 pts/3    00:00:00 grep mongod
[root@node3 conf]# 
[root@node3 conf]# service mongod stop
Stopping mongodb:                                          [  OK  ]
```

#### 2.1.3、`db.shutdownServer()`方式 <a href="#213dbshutdownserver-fang-shi" id="213dbshutdownserver-fang-shi"></a>

```bash
# mongo localhost:27000
> use admin
> db.shutdownServer()
```

#### 2.1.4、使用命令行方式关闭(补充@20160901) <a href="#214-shi-yong-ming-ling-hang-fang-shi-guan-bi-bu-chong-20160901" id="214-shi-yong-ming-ling-hang-fang-shi-guan-bi-bu-chong-20160901"></a>

```bash
# mongod -f /etc/mongo-m.conf  --shutdown
```

#### 2.1.5、强制关闭`mongod` <a href="#215-qiang-zhi-guan-bi-mongod" id="215-qiang-zhi-guan-bi-mongod"></a>

```bash
# kill -9 5675

缺点：
数据库直接关闭
数据丢失
数据文件容易损坏(需要进行修复)
```
