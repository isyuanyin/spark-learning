## Hadoop 编译

参考博客：https://www.cnblogs.com/qingyunzong/p/8494990.html



### 安装

主要参考 BUILDING.txt 中的说明。

Requirements：

```
Requirements:

* Unix System
* JDK 1.7 or 1.8
* Maven 3.0 or later
* Findbugs 1.3.9 (if running findbugs)
* ProtocolBuffer 2.5.0
* CMake 2.6 or newer (if compiling native code), must be 3.0 or newer on Mac
* Zlib devel (if compiling native code)
* openssl devel (if compiling native hadoop-pipes and to get the best HDFS encryption performance)
* Linux FUSE (Filesystem in Userspace) version 2.6 or above (if compiling fuse_dfs)
* Internet connection for first build (to fetch all Maven and Hadoop dependencies)
* python (for releasedocs)
* Node.js / bower / Ember-cli (for YARN UI v2 building)
```



* 这里先安装好 oracle java8，部署 hadoop 时已经装好，修改 /etc/profile 文件，然后 source 该文件。

增加如下：

```shell
export JAVA_HOME=/usr/local/jvm/jdk1.8.0_261
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=$PATH:${JAVA_HOME}/bin
```



* 然后卸载openjdk，安装maven 

```shell
  $ sudo apt-get purge openjdk*  # 清除openjdk
  $ sudo apt-get -y install maven、
  
  $ mvn -v
```

这里可能要重启电脑再安装maven，openjdk有点难缠。最后版本如下：

```
Apache Maven 3.6.3
Maven home: /usr/share/maven
Java version: 1.8.0_261, vendor: Oracle Corporation, runtime: /usr/local/jvm/jdk1.8.0_261/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "linux", version: "5.4.0-42-generic", arch: "amd64", family: "unix"
```



* 安装各种本地库 和 protocal buffer

```shell
  $ sudo apt-get -y install build-essential autoconf automake libtool cmake zlib1g-dev pkg-config libssl-dev
  
  $ sudo apt-get update
  $ sudo apt-get -y install protobuf-compiler
```

查看安装结果：

```shell
hadoop@master:~/Desktop$ protoc --version
libprotoc 3.6.1
```



* 以下时可选的包

```
Optional packages:

* Snappy compression
  $ sudo apt-get install snappy libsnappy-dev
* Bzip2
  $ sudo apt-get install bzip2 libbz2-dev
* Jansson (C Library for JSON)
  $ sudo apt-get install libjansson-dev
* Linux FUSE
  $ sudo apt-get install fuse libfuse-dev
```





### 编译

设置maven使用的内存大小：

```shell
export MAVEN_OPTS="-Xms256m -Xmx1536m"
# Here is an example setting to allocate between 256 MB and 1.5 GB of heap space to Maven
```



编译:

```shell
 # Create binary distribution with native code and with documentation
 $ mvn package -Pdist,native,docs -DskipTests -Dtar
```



### 出错

* protocal buffer 版本过新

```shell
'libprotoc 3.6.1', expected version is '2.5.0'
```

重新下载安装protocal buffer 2.5.0，网址：https://github.com/protocolbuffers/protobuf/releases/tag/v2.5.0

```shell
sudo su  #进入root用户
chmod 755 protobuf-2.5.0.tar.gz 
tar -zxvf protobuf-2.5.0.tar.gz -C /opt
cd /opt/protobuf-2.5.0/
./configure 
make
make install
```

输入 ：

```shell
protoc --version
```

出错：

```
protoc: error while loading shared libraries: libprotoc.so.8: cannot open shared
```

解决方法：

```shell
sudo ldconfig
# or
export LD_LIBRARY_PATH=/usr/local/lib
```



* 找不到 tools.jar

```
[ERROR] Artifact: jdk.tools:jdk.tools:jar:1.8 has no file.
```



```
https://repo.maven.apache.org/maven2/org/apache/maven/plugins/maven-compiler-plugin/3.8.1/maven-compiler-plugin-3.8.1.pom
```







### 安装的包说明

安装的包：

* maven：编译工具

* protocol buffer：是 Google 公司内部的混合语言数据标准

依赖的包

* build-essential：作用是提供编译程序必须软件包的列表信息，安装了该软件包，编译c/c++所需要的软件包也都会被安装。

* autoconf：用于生成shell脚本的工具

* automake：一个从文件`Makefile.am`自动生成`Makefile.in` 的工具

* libtool：是一个通用库支持脚本，将使用动态库的复杂性隐藏在统一、可移植的接口中，也就是说，你可以通过如下所示的标准方法，在不同平台上创建并调用动态库，我们可以认为libtool是gcc的一个抽象，也就是说，它包装了gcc或者其他的任何编译器，用户无需知道细节，只要告诉libtool说我需要要编译哪些库即可，并且它只与libtool文件打交道，例如lo、la为后缀的文件。

* cmake：跨平台的编译工具

* zlib1g-dev ：提供数据压缩用的函式库

* pkg-config：向用户向程序提供相应库的路径、版本号等信息的程序。

* libssl-dev：SSL安装

