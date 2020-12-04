## Hadoop 部署

参考博客：https://www.cnblogs.com/qingyunzong/p/8496127.html

部署环境：

* VMWare 虚拟机软件

* 4个 ubuntu20.04 桌面系统（简易安装，用户名相同为hadoop）

* Xshell（ssh远程访问软件）

* 软件工具：Java1.8.0_261；Hadoop2.10.0



### 开启 ssh 远程登录

```shell
sudo apt-get install net-tools      # 安装net套件
ifconfig                            # 查看ip地址

sudo apt-get install openssh-server
sudo service ssh start
```



文件传输：为了实现简单的文件传输，需要在虚拟机的 Linux 系统里安装一个小工具 lrzsz

```shell
sudo apt-get update
sudo apt-get install lrzsz
```

lrzsz 是一款在 Linux 里可代替 ftp 上传和下载的程序，安装后在终端里输入 rz回车，就可以在弹出的窗口中选择本地文件上传到远程主机的当前目录下，而输入 `sz <filename>` 就可以把远程主机里的文件下载到本地。



### ubuntu20 设置ip

进入 /etc/netplan/，编辑yaml文件

```shell
cd /etc/netplan
sudo vi 01-network-manager-all.yaml 
# 或者
sudo gedit /etc/netplan/01-network-manager-all.yaml 
```

输入内容：

```yaml
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens33:
      dhcp4: false
      addresses: [192.168.149.140/24]
      gateway4: 192.168.149.2
      nameservers:    
        addresses: [192.168.149.2,8.8.8.8,114.114.114.114]
```

保存后，输入：

```shell
sudo netplan --debug apply
```

然后输入ifconfig可以看到ip地址已经修改。



### 机器节点互通

参考博客：https://www.jianshu.com/p/e22058f6ecbf

* 修改主机名

```shell
sudo vi /etc/hostname   #编辑 /etc/hostname 文件从而修改主机名
sudo reboot             #重启使新主机名生效
```



* 修改各个 hosts 文件，在本地植入部分 DNS 映射，将对应的角色名与 IP 匹配起来，然后尝试用角色名相互 ping，相互能 ping 通证明配置成功：

```shell
sudo vi /etc/hosts    #编辑 /etc/hosts 文件，插入角色与 IP 映射
ping master -c 4      #尝试用角色名 ping 其它主机，一次 4 个包
```

​		其中设置如下：

```shell
127.0.0.1	localhost
127.0.1.1	ubuntu

192.168.149.140 master
192.168.149.141 slave1
192.168.149.142 slave2
192.168.149.143 slave3

# The following lines are desirable for IPv6 capable hosts
```



* 配置 SSH 无密码登录

```shell
cd ~/.ssh            # 如果没有该目录，先执行一次 ssh localhost
rm ./id_rsa*         # 删除之前生成的公匙（如果有）
ssh-keygen -t rsa    # 一直按回车就可以
```



### 安装 Java

* 将上传的 JDK 压缩包（jdk-8u60-linux-x64.tar）放到家目录（/home/hadoop/），解压并放到指定的文件夹：

```shell
sudo mkdir -p /usr/local/jvm
sudo tar -zxvf jdk-8u261-linux-x64.tar.gz -C /usr/local/jvm
```



* 将当前的 PATH 环境变量提取保存到 setenv.sh，然后将其修改为初始化语句，增加JAVA 的路径：

```shell
echo $PATH >> ~/setenv.sh
vi ~/setenv.sh
```

​		修改的内容：

```shell
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

export JAVA_HOME=/usr/local/jvm/jdk1.8.0_261
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=$PATH:${JAVA_HOME}/bin
```

*  

```shell
source ~/setenv.sh
java -version
javac -version
```



### 安装配置 Hadoop

* 安装hadoop

```shell
tar -zxvf hadoop-2.10.0.tar.gz
sudo mv hadoop-2.10.0 /usr/local/hadoop #mv 实现重命名
sudo chown -R hadoop:hadoop /usr/local/hadoop
```



需要配置的文件包括 slaves; core-site.xml; hdfs-site.xml; mapred-site.xml; yarn-site.xml; hadoop-env.sh

* 修改 slaves 文件，让 hadoop 知道自己可以聚合的节点名（保证与 hosts 里的角色名一致）：

```shell
gedit /usr/local/hadoop/etc/hadoop/slaves
```

```
master
slave1
slave2
slave3
```



* 修改 core-site.xml 文件如下：

```
gedit /usr/local/hadoop/etc/hadoop/core-site.xml
```

配置信息：

```xml
<configuration>
	<property>
		<name>fs.default.name</name>
		<value>hdfs://master:9000</value>
	</property>
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/usr/local/hadoop/tmp</value>
	</property>	
</configuration>
```



* 修改 hdfs-site.xml 文件如下（启用所有节点作为 DataNode，故 replication=3）

```shell
gedit /usr/local/hadoop/etc/hadoop/hdfs-site.xml
```

配置信息：

```xml
<configuration>

	<property>
		<name>dfs.replication</name>
		<value>3</value>
	</property>
	<property>
		<name>dfs.name.dir</name>
		<value>/usr/local/hadoop/hdfs/name</value>
	</property>	
	<property>
		<name>dfs.data.dir</name>
		<value>/usr/local/hadoop/hdfs/data</value>
	</property>	
	<property>
		<name>dfs.secondary.http.address</name>
		<value>slave3.50090</value>
	</property>
</configuration>
```



* 修改 mapred-site.xml 设置

```shell
cd /usr/local/hadoop/etc/hadoop/
cp mapred-site.xml.template mapred-site.xml
gedit /usr/local/hadoop/etc/hadoop/mapred-site.xml
```

配置信息：

```xml
<configuration>
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
</configuration>
```



* 修改 yarn-site.xml 设置

```shell
 gedit /usr/local/hadoop/etc/hadoop/yarn-site.xml
```

配置信息：

```xml
        <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>

        <property>
                <name>yarn.resourcemanager.hostname</name>
                <value>master</value>
        </property>
```

注：这里将 resource manager 放在master结点上，也可以放在其它结点。



* 修改 hadoop-env.sh 文件，将 25 行 JAVA_HOME 的值换成 jdk 所在的路径：

```
gedit /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```

配置信息：

```shell
export JAVA_HOME=/usr/local/jvm/jdk1.8.0_261
```



### 启动(关闭) HDFS 和 Yarn

a) Namenode 格式化：

```shell
/usr/local/hadoop/bin/hdfs namenode -format
```



b) 启动：

```shell
/usr/local/hadoop/sbin/start-dfs.sh
/usr/local/hadoop/sbin/start-yarn.sh

jps # 每个节点查看一遍
```

c) 关闭：

```
cd /usr/local/hadoop/sbin/
stop-yarn.sh
stop-dfs.sh 

stop-all.sh
```

一键启动关闭（dfs和yarn）：

```shell
start-all.sh
stop-all.sh
```



* 测试 Hadoop

在/usr/local/hadoop目录下

a) 在 hdfs 上创建输入文件夹 input，并把 etc/hadoop 下的所有文本文件放进去

```shell
./bin/hdfs dfs -mkdir /input
./bin/hdfs dfs -put etc/hadoop/*.xml /input
```

b) 测试样例

```shell
./bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.10.0.jar wordcount /input /output
```





