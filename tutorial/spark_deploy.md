## Spark 部署

部署环境：

* hadoop-2.10.0

* spark-2.4.6-bin-without-hadoop



### Spark 安装

* 解压（下面只在master结点执行）

```shell
sudo tar -xzvf spark-2.4.6-bin-without-hadoop.tgz
sudo mv spark-2.4.6-bin-without-hadoop /usr/local/spark
sudo chown -R hadoop:hadoop /usr/local/spark/
```



* 生成配置文件

```shell
cp /usr/local/spark/conf/spark-env.sh.template /usr/local/spark/conf/spark-env.sh
cp /usr/local/spark/conf/slaves.template /usr/local/spark/conf/slaves
cp /usr/local/spark/conf/spark-defaults.conf.template /usr/local/spark/conf/spark-defaults.conf
```





* 修改 spark-env.sh



```shell
export HADOOP_HOME=/usr/local/hadoop
export SPARK_HOME=/usr/local/spark
export JAVA_HOME=/usr/local/jvm/jdk1.8.0_261
export SPARK_CONF_DIR=${SPARK_HOME}/conf
export SPARK_DIST_CLASSPATH=$(/usr/local/hadoop/bin/hadoop classpath)   # path of hadoop jar
export HADOOP_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export YARN_CONF_DIR=${HADOOP_HOME}/etc/hadoop
export SPARK_MASTER_IP=master
export SPARK_LOCAL_DIRS=/usr/local/spark
export SPARK_WORKER_MEMORY=2000M
export SPARK_EXECUTOR_MEMORY=2000M
export SPARK_DRIVER_MEMORY=2000M
export SPARK_EXECUTOR_CORES=2
```



* 修改 slaves 文件，在文件末尾添加其他节点 IP

```
gedit /usr/local/spark/conf/slaves
```

修改信息：

```shell
master
slave1
slave2
slave2
```



* 修改 spark-defaults.conf，在文件末尾添加如下内容：

```
spark.executor.extraJavaOptions -XX:+PrintGCDetails -Dkey=value -Dnumbers="one two three"
spark.eventLog.enabled true
spark.eventLog.dir hdfs://master:9000/historyserverforSpark
spark.yarn.historyServer.address master:18080
spark.history.fs.logDirectory hdfs://master:9000/historyserverforSpark
spark.speculation true
```



* 将配置好的 spark 发送的其它结点

```shell
scp -r /usr/local/spark hadoop@slave1:/usr/local/spark
```

注：可能不行，可以先发到 /home/hadoop/ 中



### 配置 hadoop

* 修改 yarn-site.xml 文件，添加新的属性。

```shell
        <property>
                <name>yarn.log-aggregation-enable</name>
                <value>true</value>
        </property>
```



### 运行 Spark

1、运行 hadoop

```shell
/usr/local/hadoop/sbin/start-all.sh
```

2、在 spark 中创建 historyserverforSpark 文件夹

```shell
/usr/local/hadoop/bin/hdfs dfs -mkdir historyserverforSpark
```

3、运行 spark

```shell
/usr/local/spark/sbin/start-all.sh
```

可以进入 spark 的 webui 查看是否成功启动：master://8080/  (这里master要换成具体ip)

![image-20200728101148319](images\image-20200728101148319.png)



4、运行 history-server，这样应用运行完的结果可以通过 webui 看到。

<img src="images\image-20200728101238048.png" alt="image-20200728101238048"  />