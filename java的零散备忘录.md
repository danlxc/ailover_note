
3 +4（外派）+ 5 

VMware 14 Pro 永久许可证激活密钥
FF31K-AHZD1-H8ETZ-8WWEZ-WUUVA
CV7T2-6WY5Q-48EWP-ZXY7X-QGUWD

NOD32 v11/10/9 许可证密钥有效激活码
CNDU-W33E-59SK-K2BF-9JST 到期日2019-10-14
CNDU-W337-WAFE-EUU9-F55M 到期日2019-03-13

192.168.136.94

sudo netstat -tap | grep mysql

bbb111
127.0.0.1
::1

192.168.177.177
nohup
 df 的 -h 参数以可读性较高的方式来显示信息。
 uptime  系统运行负载情况
 su root  切换用户：su 
 free -m 查看内存使用情况 以 m 为单位显示内存大小
 reboot 重启系统
 kill pid：杀掉进程
 kill -9 pid 强制杀掉进程
 shutdown -h now 关机
 mkdir -pv /home/dir/{a..e} (-p表示递归 -v表示查看创建详情)
 touch 创建文件
 mkdir -pv  /usr/local/cluster/activemq/ && touch /usr/local/cluster/activemq/kahadb

-bin ./zkServer.sh start 
-bin ./activemq start (./activemq stop)
-bin ./mysqld_safe & (./mysqladmin -uroot -p shutdown)
(mysql 集群)
-bin ./mysqld_safe --defaults-file=/usr/local/mysql-5.9.18/data/3307/my.cnf & 
-bin ./startup.sh | tail -f ../logs/catalina.out (./shutdown.sh) tomcat

-src ./redis-sentinel ../sentinel26384.conf & 
-src ./redis-server ../redis.conf & (./redis-cli shutdown) -src

-sbin ./nginx -c /usr/local/nginx/conf/nginx.conf
(kill -QUIT 主pid(优雅)kill -TERM 主pid(快速)kill -HUP 主pid(平滑重启))
fdfs_trackerd /etc/fdfs/tracker.conf
fdfs_storaged /etc/fdfs/storage.conf


） 重 定向输出追加 ： >>
echo “hello new word” >> t1.txt

语法: tar 参数 要压缩或解压的文件或目录
常用参数：
z : 使用 gzip 解压缩程序，生成的文件名是 xxx.tar.gz 这是 linux 中常用的压缩格式。
c : 创建压缩文档
v : 显示压缩，解压过程中处理的文件名
f : 指定归档文件名, tar 参数后面是归档文件名
x : 从归档文件中释放文件，就是解压。
t : 列出归档文件内容，查看文件内容
C: 解压到指定目录，使用方式 -C 目录

 查看归档/压缩语法：tar -tf 归档文件名
 解压语法tar -zxvf file.tar.gz -C /usr/local
 解压归档语法：tar -xvf 归档文件名
 压缩语法：tar -zcvf 归档文件名 要归档文件列表/目录
 不压缩回档语法：tar -cvf 归档文件名.tar 文件列表/目录 ， 就是之前的命令去掉 z ,不压缩只归档。


redis 集群启动 (以下所有的都不用使用& 来后台启动, 因为在配置文件的时候有一个属性表示后台启动(daemonize yes),
但是sentinel26384 我没有设置,因为要用来显示日志信息)
./redis-server ../redis6380.conf   master
./redis-server ../redis6382.conf 
./redis-server ../redis6384.conf 

redis-sentinel /usr/local/redis-3.2.9/sentinel26380.conf  
redis-sentinel /usr/local/redis-3.2.9/sentinel26382.conf 
redis-sentinel /usr/local/redis-3.2.9/sentinel26384.conf

哨兵模式配置
daemonize yes
port 26380
dir "/tmp"
sentinel myid ac85c8b7f3fcf86e0fa8857acb177e68b97168b1
sentinel monitor mymaster 192.168.177.177 6382 2
sentinel auth-pass mymaster danny
sentinel config-epoch mymaster 9
sentinel leader-epoch mymaster 9
protected-mode no

zookeeper集群
/usr/local/zookeeper-3.4.10_1/bin/zkServer.sh start
/usr/local/zookeeper-3.4.10_2/bin/zkServer.sh start
/usr/local/zookeeper-3.4.10_3/bin/zkServer.sh start
./zkServer.sh status 查看状态

集群配置文件  还有别忘配置myid 文件 对应下面的1,2,3
Zookeeper-3.4.10_1 的 zoo.cfg:
dataDir=/usr/local/zookeeper-3.4.10_1/data/
clientPort=2182
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890

systemctl stop firewalld
systemctl status firewalld

通过上述命令检查之后，如果看到有mysql 的socket处于 listen 状态则表示安装成功。

http://download.oracle.com/otn-pub/java/jdk/8u144-b01/090f390dda5b47b9b721c7dfaa008135/jdk-8u144-linux-i586.tar.gz

http://219.238.7.73/files/216300000A20F295/download.oracle.com/otn/java/jdk/8u121-b13/e9e7ea248e2c4826b92b3f075a80e441/jdk-8u121-linux-x64.tar.gz

export JAVA_HOME=/usr/local/jdk1.8.0_121
export PATH=$JAVA_HOME/bin:$PATH
export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JAVA_HOME/jre/lib/rt.jar

./mysqld  --initialize  --user=mysql  --datadir=/usr/local/mysql-5.9.18/data --basedir=/usr/local/mysql-5.9.18

root@localhost: _xqMXeWd_0(y



B 、  编辑 Master  配置文件
编辑 Master 的配置文件 redis6380.conf : 在空文件加入如下内容
include /usr/local/redis-3.2.9/redis.conf
daemonize yes
port 6380
pidfile /var/run/redis_6380.pid
logfile 6380.log
dbfilename dump6380.rdb
配置项说明：
include ： 包含原来的配置文件内容。/usr/local/redis-3.2.9/redis.conf 按照自己的目录设置。
daemonize：yes 后台启动应用，相当于 ./redis-server & , &的作用。
port : 自定义的端口号
pidfile : 自定义的文件，表示当前程序的 pid ,进程 id。
logfile：日志文件名
dbfilename：持久化的 rdb 文件名
C 、  编辑 Slave  配置文件
编辑 Slave 的配置文件 redis6382.conf 和 redis6384.conf: 在空文件加入如下内容
①：redis6382.conf：
include /usr/local/redis-3.2.9/redis.conf
daemonize yes
port 6382
pidfile /var/run/redis_6382.pid
logfile 6382.log
dbfilename dump6382.rdb
slaveof 127.0.0.1 6380
配置项说明：
slaveof ： 表示当前 Redis 是谁的从。当前是 127.0.0.0 端口 6380 这个 Master 的从。

（2 ） 份 三份 sentinel  配置文件修改：
1、修改 port 26380、 port 26382、 port 26384
2、修改 sentinel monitor mymaster 127.0.0.1 6380 2
格式：Sentinel monitor <name> <masterIP> <masterPort><Quorum 投票数>
Sentinel监控主(Master)Redis, Sentinel根据Master的配置自动发现Master的Slave,Sentinel
默认端口号为26379 。

 (zookeeper) 改 修改 zoo.conf
加入集群中各 Zookeeper 的信息，3 个 Zookeeper 各自 conf 的 zoo.cfg 文件修改内容。
参考 Zookeeper 文档，连接如下：
http://zookeeper.apache.org/doc/trunk/zookeeperStarted.html#sc_RunningReplicatedZooKeeper
Zookeeper-3.4.10_1 的 zoo.cfg:
dataDir=/usr/local/zookeeper-3.4.10_1/data/
clientPort=2182
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890
其中 2888 端口号是 zookeeper 服务之间通信的端口，3888 是 zookeeper 与选举新的领导者，
追随者连接到领导者服务器的端口。server.1 中数字是集群中 Zookeeper 的标志。用来区别
其他 Zookeeper。
其他两个 Zookeeper 的 conf/zoo.cfg
zookeeper-3.4.10_2/conf/zoo.cfg
clientPort=2183
dataDir=/usr/local/zookeeper-3.4.10_2/data/
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890
zookeeper-3.4.10_3/conf/zoo.cfg
clientPort=2184
dataDir=/usr/local/zookeeper-3.4.10_3/data/
server.1=localhost:2888:3888
server.2=localhost:2889:3889
server.3=localhost:2890:3890
（4 ） 加 增加 myid  文件
Zookpeer使用myid识别当前服务器是谁，和zoo.cfg中的server.1 , server.2,server.3对应。
在每个 Zookeeper 数据存放目录 data 下新建 myid 文件。文件没有扩展名。
zookeeper-3.4.10_1/data/myid 文件内容是 1
zookeeper-3.4.10_2/data/myid 文件内容是 2
zookeeper-3.4.10_3/data/myid 文件内容是 3

mysql 集群
stop slave;
reset slave;
reset master;
show master status;
设置从服务器3308、3309，他们的主均为3307，即在3308和3309上执行如下操作：
change master to master_host='192.168.177.177',
master_user='copy',
master_password='123456',
master_port=3307, 
master_log_file='mysql-bin.000001',
master_log_pos=154;

私钥
MIICdAIBADANBgkqhkiG9w0BAQEFAASCAl4wggJaAgEAAoGBAKMyNchWlgjOhP4bsa3Mjoq6NkhaLmWbQ4CPIz2FBhtyRNf3pudSu1wQgfRKsKrTj1rZwp7hIgRlMPGBb76QX6aTSQLvnzeYBT0DAppihcEdgu37/+U1vKVcDtLs0jJ4xvc00WwpbVBDCFP9Ref5OH4dLDIscUaAWWdh6AhaZLjVAgMBAAECgYAceiqAZvuPVdpHLTX4CfXlp1DJl1L5T/qbeF7B4XCLYYk51nE9dGZVTlwe8NmbNYeSZuVbLBXvhmjf+6IwMqk99Jm/8c56DS/jvuP+lg8AC1gNebC0Nbtx2+IkjZN0poU7S149Ix5QRqc6LeXUoJRSQYx1rzl7CcOj2+yYOUiPkQJBANSdfKRRhLPUO2GBt7P1PBmncWTwy7HKjELMLZBVXLsG5b5Wr9E1WUQU8JMZ7lwUa2UFFlpUKHkwEZjYD4hgdf8CQQDEfzBu+fr65D+IFFPoA81m43AD5J1EMx1Sej3bdSyNcfJ3ycoxdZVIvopALSwPu7GGToY0F1YCLT0U5OYnVRkrAkAvpPLvZ40TNzXvTcA6xXOoVAtnEUa0Gq1/sn1rYJWdG5iUJJzVhtzwErkuXZs6ayD9zDwMFdvT/F+VHqWsa+FFAkB0MEETXf1qHUzzyhFTP+xUymeR5byYdyD/hAjPm5mciaQ18Lv+Quji+sgE6rEDHJj8MbJpWuMgpl9X24G8ADXBAj9vTVYO2+okoy5MPLHGDftIUmJUMdoIRv+DVS37kHTnVoORWK1bBg7Fr6Au0Z5sqkKUDr7/krPm/lliAVv1PTs=


公钥
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCjMjXIVpYIzoT+G7GtzI6KujZIWi5lm0OAjyM9hQYbckTX96bnUrtcEIH0SrCq049a2cKe4SIEZTDxgW++kF+mk0kC7583mAU9AwKaYoXBHYLt+//lNbylXA7S7NIyeMb3NNFsKW1QQwhT/UXn+Th+HSwyLHFGgFlnYegIWmS41QIDAQAB