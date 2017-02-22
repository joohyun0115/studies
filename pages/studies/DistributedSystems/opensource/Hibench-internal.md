#Prerequisites

## Environment
Ubuntu 14.04 64bit

##Install Sun jdk
sudo apt-get install software-properties-common
sudo apt-add-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java7-installer

java -version

kesl@kesl-All-Series:~/github$ java -version
java version "1.7.0_67"
Java(TM) SE Runtime Environment (build 1.7.0_67-b01)
Java HotSpot(TM) 64-Bit Server VM (build 24.65-b04, mixed mode)

##Install some package
sudo apt-get install -y libprotobuf-dev protobuf-compiler maven cmake \
build-essential pkg-config libssl-dev zlib1g-dev llvm-gcc-4.8 automake \
autoconf make

##Install scala 
wget http://www.scala-lang.org/files/archive/scala-2.10.4.tgz
sudo mkdir /usr/local/src/scala
sudo tar xvf scala-2.10.4.tgz -C /usr/local/src/scala/

vi .bashrc
export SCALA_HOME=/usr/local/src/scala/scala-2.10.4
export PATH=$SCALA_HOME/bin:$PATH


kesl@kesl-All-Series:~/github$ scala -version
Scala code runner version 2.10.4 -- Copyright 2002-2013, LAMP/EPFL


##Install hadoop

http://www.bogotobogo.com/Hadoop/BigData_hadoop_Install_on_ubuntu_single_node_cluster.php
ref: https://www.youtube.com/watch?v=D5h1kGBwAsA

```bash
wget http://mirrors.sonic.net/apache/hadoop/common/hadoop-2.6.0/hadoop-2.6.0.tar.gz
tar xvzf hadoop-2.6.0.tar.gz
sudo mv hadoop-2.6.0/ /usr/local/
cd /usr/local/
sudo ln -s hadoop-2.6.0/ hadoop
```

sudo addgroup hadoop
sudo adduser --ingroup hadoop hduser

Adding user `hduser' ...
Adding new user `hduser' (1002) with group `hadoop' ...
Creating home directory `/home/hduser' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for hduser
Enter the new value, or press ENTER for the default
	Full Name []: 
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] y

su hduser
ssh-keygen -t rsa -P ""

Generating public/private rsa key pair.
Enter file in which to save the key (/home/hduser/.ssh/id_rsa): 
Created directory '/home/hduser/.ssh'.
Your identification has been saved in /home/hduser/.ssh/id_rsa.
Your public key has been saved in /home/hduser/.ssh/id_rsa.pub.
The key fingerprint is:
a1:1b:b0:c1:5b:1b:30:dc:43:44:f7:51:01:60:16:fa hduser@kesl-All-Series
The key's randomart image is:
+--[ RSA 2048]----+
|   .o=+ *oooo.   |
|   ..oo= . .     |
|    + +.. .      |
|     * = .       |
|    o + E        |
|       o         |
|      .          |
|                 |
|                 |
+-----------------+

cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys

hduser@kesl-All-Series:/usr/local$ ssh localhost
The authenticity of host 'localhost (127.0.0.1)' can't be established.
ECDSA key fingerprint is a5:2b:21:6f:c5:40:ff:30:cc:78:db:4c:7d:e1:ba:a4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

exit
exit
sudo adduser hduser sudo

sudo su hduser

sudo chown -R hduser:hadoop /usr/local/hadoop

The following files will have to be modified to complete the Hadoop setup:

~/.bashrc
/usr/local/hadoop/etc/hadoop/hadoop-env.sh
/usr/local/hadoop/etc/hadoop/core-site.xml
/usr/local/hadoop/etc/hadoop/mapred-site.xml.template
/usr/local/hadoop/etc/hadoop/hdfs-site.xml

hduser@laptop:~$ vi ~/.bashrc
'''bash
export JAVA_HOME=/usr/lib/jvm/java-7-oracle
export HADOOP_INSTALL=/usr/local/hadoop
export PATH=$PATH:$HADOOP_INSTALL/bin
export PATH=$PATH:$HADOOP_INSTALL/sbin
export HADOOP_MAPRED_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_HOME=$HADOOP_INSTALL
export HADOOP_HDFS_HOME=$HADOOP_INSTALL
export YARN_HOME=$HADOOP_INSTALL
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_INSTALL/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_INSTALL/lib"
'''

hduser@laptop:~$ source ~/.bashrc

hduser@kesl-All-Series:/usr/local$ javac -version
javac 1.7.0_80
hduser@kesl-All-Series:/usr/local$ readlink -f /usr/bin/javac /usr/lib/jvm/java-7-oracle/bin/javac

vi /usr/local/hadoop/etc/hadoop/hadoop-env.sh

hduser@laptop:~$ vi /usr/local/hadoop/etc/hadoop/hadoop-env.sh

export JAVA_HOME=/usr/lib/jvm/java-7-oracle

hduser@kesl-All-Series:/usr/local$ sudo mkdir -p /app/hadoop/tmp
hduser@kesl-All-Series:/usr/local$ sudo chown hduser:hadoop /app/hadoop/tmp

hduser@kesl-All-Series:/usr/local$ vi /usr/local/hadoop/etc/hadoop/core-site.xml

<configuration>
 <property>
  <name>hadoop.tmp.dir</name>
  <value>/app/hadoop/tmp</value>
  <description>A base for other temporary directories.</description>
 </property>

 <property>
  <name>fs.default.name</name>
  <value>hdfs://localhost:54310</value>
  <description>The name of the default file system.  A URI whose
  scheme and authority determine the FileSystem implementation.  The
  uri's scheme determines the config property (fs.SCHEME.impl) naming
  the FileSystem implementation class.  The uri's authority is used to
  determine the host, port, etc. for a filesystem.</description>
 </property>
</configuration>

hduser@kesl-All-Series:/usr/local$ cp /usr/local/hadoop/etc/hadoop/mapred-site.xml.template /usr/local/hadoop/etc/hadoop/mapred-site.xml

hduser@kesl-All-Series:/usr/local$ vi /usr/local/hadoop/etc/hadoop/mapred-site.xml

<configuration>
 <property>
  <name>mapred.job.tracker</name>
  <value>localhost:54311</value>
  <description>The host and port that the MapReduce job tracker runs
  at.  If "local", then jobs are run in-process as a single map
  and reduce task.
  </description>
 </property>
</configuration>

hduser@kesl-All-Series:/usr/local$ sudo mkdir -p /usr/local/hadoop_store/hdfs/namenode
hduser@kesl-All-Series:/usr/local$ sudo mkdir -p /usr/local/hadoop_store/hdfs/datanode
hduser@kesl-All-Series:/usr/local$ sudo chown -R hduser:hadoop /usr/local/hadoop_store

hduser@kesl-All-Series:/usr/local$ vi /usr/local/hadoop/etc/hadoop/hdfs-site.xml
<configuration>
 <property>
  <name>dfs.replication</name>
  <value>1</value>
  <description>Default block replication.
  The actual number of replications can be specified when the file is created.
  The default is used if replication is not specified in create time.
  </description>
 </property>
 <property>
   <name>dfs.namenode.name.dir</name>
   <value>file:/usr/local/hadoop_store/hdfs/namenode</value>
 </property>
 <property>
   <name>dfs.datanode.data.dir</name>
   <value>file:/usr/local/hadoop_store/hdfs/datanode</value>
 </property>
</configuration>

hduser@kesl-All-Series:/usr/local$ hadoop namenode -format
STARTUP_MSG:   build = https://git-wip-us.apache.org/repos/asf/hadoop.git -r e3496499ecb8d220fba99dc5ed4c99c8f9e33bb1; compiled by 'jenkins' on 2014-11-13T21:10Z
STARTUP_MSG:   java = 1.7.0_80
************************************************************/
16/01/26 15:57:16 INFO namenode.NameNode: registered UNIX signal handlers for [TERM, HUP, INT]
16/01/26 15:57:16 INFO namenode.NameNode: createNameNode [-format]
16/01/26 15:57:17 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Formatting using clusterid: CID-0d71aecf-4076-4c2a-bd71-6d4f40215195
16/01/26 15:57:17 INFO namenode.FSNamesystem: No KeyProvider found.
16/01/26 15:57:17 INFO namenode.FSNamesystem: fsLock is fair:true
16/01/26 15:57:17 INFO blockmanagement.DatanodeManager: dfs.block.invalidate.limit=1000
16/01/26 15:57:17 INFO blockmanagement.DatanodeManager: dfs.namenode.datanode.registration.ip-hostname-check=true
16/01/26 15:57:17 INFO blockmanagement.BlockManager: dfs.namenode.startup.delay.block.deletion.sec is set to 000:00:00:00.000
16/01/26 15:57:17 INFO blockmanagement.BlockManager: The block deletion will start around 2016 Jan 26 15:57:17
16/01/26 15:57:17 INFO util.GSet: Computing capacity for map BlocksMap
16/01/26 15:57:17 INFO util.GSet: VM type       = 64-bit
16/01/26 15:57:17 INFO util.GSet: 2.0% max memory 889 MB = 17.8 MB
16/01/26 15:57:17 INFO util.GSet: capacity      = 2^21 = 2097152 entries
16/01/26 15:57:17 INFO blockmanagement.BlockManager: dfs.block.access.token.enable=false
16/01/26 15:57:17 INFO blockmanagement.BlockManager: defaultReplication         = 1
16/01/26 15:57:17 INFO blockmanagement.BlockManager: maxReplication             = 512
16/01/26 15:57:17 INFO blockmanagement.BlockManager: minReplication             = 1
16/01/26 15:57:17 INFO blockmanagement.BlockManager: maxReplicationStreams      = 2
16/01/26 15:57:17 INFO blockmanagement.BlockManager: shouldCheckForEnoughRacks  = false
16/01/26 15:57:17 INFO blockmanagement.BlockManager: replicationRecheckInterval = 3000
16/01/26 15:57:17 INFO blockmanagement.BlockManager: encryptDataTransfer        = false
16/01/26 15:57:17 INFO blockmanagement.BlockManager: maxNumBlocksToLog          = 1000
16/01/26 15:57:17 INFO namenode.FSNamesystem: fsOwner             = hduser (auth:SIMPLE)
16/01/26 15:57:17 INFO namenode.FSNamesystem: supergroup          = supergroup
16/01/26 15:57:17 INFO namenode.FSNamesystem: isPermissionEnabled = true
16/01/26 15:57:17 INFO namenode.FSNamesystem: HA Enabled: false
16/01/26 15:57:17 INFO namenode.FSNamesystem: Append Enabled: true
16/01/26 15:57:17 INFO util.GSet: Computing capacity for map INodeMap
16/01/26 15:57:17 INFO util.GSet: VM type       = 64-bit
16/01/26 15:57:17 INFO util.GSet: 1.0% max memory 889 MB = 8.9 MB
16/01/26 15:57:17 INFO util.GSet: capacity      = 2^20 = 1048576 entries
16/01/26 15:57:17 INFO namenode.NameNode: Caching file names occuring more than 10 times
16/01/26 15:57:17 INFO util.GSet: Computing capacity for map cachedBlocks
16/01/26 15:57:17 INFO util.GSet: VM type       = 64-bit
16/01/26 15:57:17 INFO util.GSet: 0.25% max memory 889 MB = 2.2 MB
16/01/26 15:57:17 INFO util.GSet: capacity      = 2^18 = 262144 entries
16/01/26 15:57:17 INFO namenode.FSNamesystem: dfs.namenode.safemode.threshold-pct = 0.9990000128746033
16/01/26 15:57:17 INFO namenode.FSNamesystem: dfs.namenode.safemode.min.datanodes = 0
16/01/26 15:57:17 INFO namenode.FSNamesystem: dfs.namenode.safemode.extension     = 30000
16/01/26 15:57:17 INFO namenode.FSNamesystem: Retry cache on namenode is enabled
16/01/26 15:57:17 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis
16/01/26 15:57:17 INFO util.GSet: Computing capacity for map NameNodeRetryCache
16/01/26 15:57:17 INFO util.GSet: VM type       = 64-bit
16/01/26 15:57:17 INFO util.GSet: 0.029999999329447746% max memory 889 MB = 273.1 KB
16/01/26 15:57:17 INFO util.GSet: capacity      = 2^15 = 32768 entries
16/01/26 15:57:17 INFO namenode.NNConf: ACLs enabled? false
16/01/26 15:57:17 INFO namenode.NNConf: XAttrs enabled? true
16/01/26 15:57:17 INFO namenode.NNConf: Maximum size of an xattr: 16384
16/01/26 15:57:17 INFO namenode.FSImage: Allocated new BlockPoolId: BP-1942035059-127.0.1.1-1453791437412
16/01/26 15:57:17 INFO common.Storage: Storage directory /usr/local/hadoop_store/hdfs/namenode has been successfully formatted.
16/01/26 15:57:17 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
16/01/26 15:57:17 INFO util.ExitUtil: Exiting with status 0
16/01/26 15:57:17 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at kesl-All-Series/127.0.1.1
************************************************************/

Starting Hadoop
hduser@kesl-All-Series:/usr/local$ cd /usr/local/hadoop/sbin
hduser@kesl-All-Series:/usr/local/hadoop/sbin$ ls
distribute-exclude.sh  hdfs-config.sh           refresh-namenodes.sh  start-balancer.sh    start-yarn.cmd  stop-balancer.sh    stop-yarn.cmd
hadoop-daemon.sh       httpfs.sh                slaves.sh             start-dfs.cmd        start-yarn.sh   stop-dfs.cmd        stop-yarn.sh
hadoop-daemons.sh      kms.sh                   start-all.cmd         start-dfs.sh         stop-all.cmd    stop-dfs.sh         yarn-daemon.sh
hdfs-config.cmd        mr-jobhistory-daemon.sh  start-all.sh          start-secure-dns.sh  stop-all.sh     stop-secure-dns.sh  yarn-daemons.sh

hduser@kesl-All-Series:/usr/local/hadoop/sbin$ start-all.sh

hduser@kesl-All-Series:/usr/local/hadoop/sbin$ jps
9925 DataNode
10415 ResourceManager
10261 SecondaryNameNode
10830 Jps
9689 NameNode
10660 NodeManager

hduser@kesl-All-Series:/usr/local/hadoop/sbin$ netstat -plten | grep java
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 0.0.0.0:50020           0.0.0.0:*               LISTEN      1002       4577845     9925/java       
tcp        0      0 127.0.0.1:54310         0.0.0.0:*               LISTEN      1002       4582424     9689/java       
tcp        0      0 0.0.0.0:50090           0.0.0.0:*               LISTEN      1002       4575876     10261/java      
tcp        0      0 0.0.0.0:50070           0.0.0.0:*               LISTEN      1002       4581539     9689/java       
tcp        0      0 0.0.0.0:50010           0.0.0.0:*               LISTEN      1002       4578812     9925/java       
tcp        0      0 0.0.0.0:50075           0.0.0.0:*               LISTEN      1002       4577841     9925/java       
tcp6       0      0 :::8040                 :::*                    LISTEN      1002       4578061     10660/java      
tcp6       0      0 :::8042                 :::*                    LISTEN      1002       4575977     10660/java      
tcp6       0      0 :::8088                 :::*                    LISTEN      1002       4586538     10415/java      
tcp6       0      0 :::35162                :::*                    LISTEN      1002       4583570     10660/java      
tcp6       0      0 :::8030                 :::*                    LISTEN      1002       4581822     10415/java      
tcp6       0      0 :::8031                 :::*                    LISTEN      1002       4581810     10415/java      
tcp6       0      0 :::8032                 :::*                    LISTEN      1002       4578067     10415/java      
tcp6       0      0 :::8033                 :::*                    LISTEN      1002       4581825     10415/java   

Stopping Hadoop
stop-all.sh

Hadoop Web Interfaces
http://localhost:50070/

##Install spark
Master development branch
git clone git://github.com/apache/spark.git

1.6 maintenance branch with stability fixes on top of Spark 1.6.0
git clone git://github.com/apache/spark.git -b branch-1.6

build/sbt -Pyarn -Phadoop-2.6 assembly

##Install thrift
$ # Install your choice of java
$ sudo apt-get install libboost-dev libboost-test-dev libboost-program-options-dev libboost-system-dev libboost-filesystem-dev libevent-dev automake libtool flex bison pkg-config g++ libssl-dev 
$ cd /tmp 
$ curl http://archive.apache.org/dist/thrift/0.9.1/thrift-0.9.1.tar.gz | tar zx 
$ cd thrift-0.9.1/ 
$ ./configure 
$ make 
$ sudo make install 
$ thrift --help

#Docker 

thrift 위치 환경 변수 수정

export HADOOP_OPTS="$HADOOP_OPTS -Djava.library.path=/usr/local/hadoop/lib/native"

spark  reference site
http://blog.cloudera.com/blog/2015/03/how-to-tune-your-apache-spark-jobs-part-1/
http://blog.cloudera.com/blog/2015/03/how-to-tune-your-apache-spark-jobs-part-2/


