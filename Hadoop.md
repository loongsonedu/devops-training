### 一、实验目的

 掌握在龙芯操作系统上部署Hadoop软件

### 二、实验准备

- 用户hadoop密码：hadoop



### 三、Hadoop软件部署

#### Step 1 – 禁用 SELinux

```
vim /etc/selinux/config
SELINUX=disabled
```

#### Step 2 – 安装 Java

```shell
java -version

[root@bogon ~]# java -version
openjdk version "1.8.0_312"
OpenJDK Runtime Environment (Loongson 8.1.10-loongarch64-LoongnixServer) (build 1.8.0_312-b07)
OpenJDK 64-Bit Server VM (build 25.312-b07, mixed mode)
```

#### Step 3 – 创建Hadoop用户

```
useradd hadoop
passwd hadoop 
Changing password for user hadoop.
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
--输入密码：hadoop
```

#### Step 4 – 配置ssh登录认证

```
su - hadoop
ssh-keygen -t rsa
提示如下：一路回车
Generating public/private rsa key pair.
Enter file in which to save the key (/home/hadoop/.ssh/id_rsa): 
Created directory '/home/hadoop/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/hadoop/.ssh/id_rsa.
Your public key has been saved in /home/hadoop/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:a/og+N3cNBssyE1ulKK95gys0POOC0dvj+Yh1dfZpf8 hadoop@centos8
The key's randomart image is:
+---[RSA 2048]----+
|                 |
|                 |
|              .  |
|     .   o o o   |
|  . . o S o o    |
| o = + O o   .   |
|o * O = B =   .  |
| + O.O.O + +   . |
|  +=*oB.+ o     E|
+----[SHA256]-----+
```

```
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 640 ~/.ssh/authorized_keys
```

#### Step 5 – 安装 Hadoop

```
su - hadoop
tar -xvzf hadoop-3.2.3.tar.gz -C /data/
mv hadoop-3.2.3 hadoop
vi  ~/.bashrc

export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-8.1.10.lns8.loongarch64/jre/
export HADOOP_HOME=/data/soft/haoop
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export HADOOP_YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib/native"
```

保存并使配置文件生效

```
source ~/.bashrc
```

修改Hadoop的配置文件

```
vi  $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

修改JAVA_HOME环境变量如下

```
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.312.b07-8.1.10.lns8.loongarch64/jre
```

#### Step 6 – 配置 Hadoop

创建Hadoop的Namenode和DataNode路径

```
mkdir -p ~/hadoopdata/hdfs/namenode
mkdir -p ~/hadoopdata/hdfs/datanode
```

更新**core-site.xml** 修改主机名

```
vim  $HADOOP_HOME/etc/hadoop/core-site.xml

<configuration>
<property>
                <name>fs.defaultFS</name>
                <value>hdfs://10.40.80.16:9000</value>
        </property>
</configuration>
```

更新**hdfs-site.xml**  修改副本数和namenode路径和datanode路径

```
<configuration>
<property>
                <name>dfs.replication</name>
                <value>1</value>
        </property>
 
        <property>
                <name>dfs.name.dir</name>
                <value>file:///home/hadoop/hadoopdata/hdfs/namenode</value>
        </property>
 
        <property>
                <name>dfs.data.dir</name>
                <value>file:////home/hadoop/hadoopdata/hdfs/datanode</value>
        </property>

</configuration>
```

更新 **mapred-site.xml** 修改任务提交框架

```
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
</configuration>
```

#### Step 7 – 启动 Hadoop Cluster

```
su  - hadoop 
hdfs namenode -format 
start-dfs.sh
start-yarn.sh
jps 验证如下进程
[hadoop@bogon hadoop]$ jps
4015002 SecondaryNameNode
4015477 ResourceManager
4028416 Jps
4014560 NameNode
4015635 NodeManager
4014754 DataNode
```

#### Step 8 – 验证集群

```shell
hdfs dfs -mkdir /test1
hdfs dfs -mkdir /test2
hdfs dfs -ls /
[hadoop@bogon hadoop]$ hdfs dfs -ls /
Found 5 items
drwxr-xr-x   - hadoop supergroup          0 2023-02-20 07:54 /input
drwxr-xr-x   - hadoop supergroup          0 2023-02-20 08:32 /output
drwxr-xr-x   - hadoop supergroup          0 2023-02-20 07:44 /test1
drwxr-xr-x   - hadoop supergroup          0 2023-02-20 07:44 /test2
drwx------   - hadoop supergroup          0 2023-02-20 08:00 /tmp

运行wordcount 程序,输出部分结果如下：
hadoop jar /data/soft/haoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-3.2.3.jar wordcount /input /output

[hadoop@bogon hadoop]$ hadoop fs -ls /output
Found 2 items
-rw-r--r--   1 hadoop supergroup          0 2023-02-20 08:32 /output/_SUCCESS
-rw-r--r--   1 hadoop supergroup         25 2023-02-20 08:32 /output/part-r-00000

[hadoop@bogon hadoop]$ hadoop fs -cat /output/part-r-00000
hadoop  1
hello   2
world   1
```

#### Step 9 – 停止 Hadoop Cluster

```
stop-dfs.sh 
stop-yarn.sh
```

