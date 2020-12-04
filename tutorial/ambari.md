# Ambari 部署 Spark

官方文档：https://cwiki.apache.org/confluence/display/AMBARI/Ambari





## Ambari 下载安装

参考：https://cwiki.apache.org/confluence/display/AMBARI/Installation+Guide+for+Ambari+2.7.5

环境：

* Ubuntu20
* JDK1.8，Maven3.6.3

### Step 1: 下载和编译 Ambari 2.7.5 source

下载源码：

```shell
wget https://mirrors.tuna.tsinghua.edu.cn/apache/ambari/ambari-2.7.5/apache-ambari-2.7.5-src.tar.gz
```

解压：

```shell
tar xfvz apache-ambari-2.7.5-src.tar.gz
```

编译：

```shell
cd apache-ambari-2.7.5-src
mvn versions:set -DnewVersion=2.7.5.0.0
 
pushd ambari-metrics
mvn versions:set -DnewVersion=2.7.5.0.0
popd
```

继续（Ubuntu）：

```shell
mvn -B clean install jdeb:jdeb -DnewVersion=2.7.5.0.0 -DbuildNumber=5895e4ed6b30a2da8a90fee2403b6cab91d19972 -DskipTests -Dpython.ver="python >= 2.6"
```



### **Step 2**: 安装 Ambari 服务

```
apt-get install ./ambari-server*.deb   #This should also pull in postgres packages as well.
```





### **Step 3**: 建立和启动 Ambari 服务

Run the setup command to configure your Ambari Server, Database, JDK, LDAP, and other options:

```
ambari-server setup
```

Follow the on-screen instructions to proceed.

Once set up is done, start Ambari Server:

```
ambari-server start
```



### **Step 4**: Install and Start Ambari Agent on All Hosts

Copy the rpm package from ambari-agent/target/rpm/ambari-agent/RPMS/x86_64/ and run:

*[Ubuntu/Debian]*

```
apt-get install ./ambari-agent*.deb
```

Edit /etc/ambari-agent/ambari.ini

```
...``[server]``hostname=localhost` `...
```

Make sure hostname under the [server] section points to the actual Ambari Server host, rather than "localhost".

```
ambari-agent start
```



### Step 5: 使用 Ambari Web UI 部署集群

打开浏览器，输入地址

```
http://<ambari-server-host>:8080.
```

* Log in with username **admin** and password **admin** and follow on-screen instructions. 

Secure your environment by ensuring your administrator details are changed from the default values as soon as possible.

Under Install Options page, enter the hosts to add to the cluster.  Do not supply any SSH key, and check "Perform manual registration on hosts and do not use SSH" and hit "Next".