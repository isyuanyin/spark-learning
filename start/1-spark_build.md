# 编译 Spark 源码

总的来说是安装 Java，Git，Maven；下载Spark源码。阅读源码时使用 Intellij IDEA，可以用学校邮箱申请免费版本 https://www.jetbrains.com/idea/。

[toc]



## 安装配置 jdk1.8

### 下载安装jdk

* 在官方网址 https://www.oracle.com/java/technologies/javase-downloads.html 下载网址找到Java SE 8u261，然后点击右边的 JDK Download。 

* 下载时可能要登陆，一个直接可使用的账号：

  ```
  2696671285@qq.com 
  密码：Oracle123
  ```

* 跳转网页后下载 jdk-8u261-windows-x64.exe，然后双击安装 java8

* 在安装过程中设置好安装路径。

Tips：jdk是包含了jvm的整套开发工具，jre就是java的运行环境，jdk包含了jre和相关的开发工具。java8之前的版本号格式是 java.1.x，比如java8的jdk版本号是jdk1.8，之后都是直接javax，比如jdk11。



### 设置环境变量

* 进入Windows的设置界面，在搜索框输入“环境变量”，然后点击“编辑账户的环境变量” （也可以点击“编辑系统...”，然后点击“环境变量”）

<img src="spark_build.assets\image-20200801111527188.png" width="80%">



* 在用户变量框中点击新建，然后输入如下（其中变量值是你安装时候的路径）：

<img src="spark_build.assets\image-20200801111737607.png" width="70%">

​	这个可以不设置，但一般都要设置。

* 然后点击其中的path路径，双击或者再点编辑，进入编辑界面。新建添加一条如下：

  ```
  D:\Program Files\Java\jdk1.8.0_261\bin
  ```

  这里可以用 %JAVA_HOME%替代路径（因为前面设置了这个变量）

  ```powershell
  %JAVA_HOME%\bin
  ```

  

Tips：

1. 环境变量是就是一个变量，可以将操作系统看成一个大型的程序，它会利用其中的变量值执行某段代码。在Linux中这个变量就是shell语言程序的变量。Windows中用 `%var%` 引用变量 `var` 的值。

2. path中的环境变量存储目录路径，控制台(cmd或者powershell)或者某些程序 在搜索文件的时候，会先在path的变量所对应的目录搜索，比如在控制台中使用 gcc 编译 .c 文件的时候，会在编译器的 bin 目录下找到 gcc.exe，所以事先要将这个 bin 的路径设置为path里面的值。
3. path是一个变量，它的变量值是若干个路径，路径直接用`;`隔开，再Linux中的 PATH 是用 `:`隔开。
4. windows用`%var%`引用变量值，而linux中的shell语言用`$var` 表示引用 var 的值。
5. 账户变量只对该电脑用户有效，系统变量对所有的电脑用户有效 (Windows 可以有多个用户)。



## 安装 Git

### 下载安装

* 在 Git 官网 https://git-scm.com/ 中点击安装 windows版本，一路默认。
* 安装成功后，在桌面右键，可以看到有 Git GUI Here 和 Git Bash Here。这个Bash中使用了linux的命令。
* 使用教程官方文档 https://git-scm.com/book/en/v2



### 基本的使用（可选）

在使用过程中新建一个目录，然后在该目录下右键，点击“Git Bash Here”就可以使用命令行。

__下载项目__

使用 git clone 就可以拉取 github 上的项目（这一步可能需要良好网络环境，可以忽略），比如：

```
git clone https://github.com/apache/spark.git
```

这个网址可以在 github 上直接复制（下图网址https://github.com/apache/spark）：

<img src="spark_build.assets\image-20200801114714898.png" width="80%">



__更新项目__

在项目的目录下：

```shell
cd spark      # 表示进入 spark 这个目录
git pull
```

__查看提交情况__

```\
git log
```



## 安装 Maven

Maven 官网：https://maven.apache.org/

### 安装 Maven

* 在官网中点击 Download，然后找到"[ apache-maven-3.6.3-bin.tar.gz](https://mirrors.bfsu.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.tar.gz)"，下载后解压，放到合适的位置。
* 将它的 bin 目录设置在账户或者系统的环境变量中。

<img src="spark_build.assets\image-20200801120025336.png" algin="left" width="50%">

### 设置下载源

Maven编译的时候要下载很多文件，可以使用国内的阿里云镜像加快下载速度。设置方式是在安装目录中修改maven的`./conf/settings.xml` 文件。

在mirrors标签内添加下面：

```xml
    <mirror>
      <id>alimaven</id>
      <mirrorOf>central</mirrorOf>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
    </mirror>
```

可能阿里云下载不了，需要中央仓库（下面不必添加）：

```xml
    <mirror>
      <id>repo1</id>
      <mirrorOf>central</mirrorOf>
      <name>central repo</name>
      <url>http://repo1.maven.org/maven2/</url>
    </mirror>
```

（还有其它阿里的库，可以查看：https://maven.aliyun.com/mvn/guide 。参考：https://zhuanlan.zhihu.com/p/71998219 ）

Tips：XML文件使用如下方式注释，可以看到mirrors标签下有一个mirror标签在注释内，不要理它。在注释外插入上面代码。

```xml
<!-- 这里是注释
	这一行还是注释
-->
```





## 编译源码

参考官网编译说明：http://spark.apache.org/docs/latest/building-spark.html，下面给出windows中的说明。

### 下载Spark源码

* 可以在spark官网中下载最新的版本，然后解压，放到合适的位置。（注意：是源码，文件名类似 spark-2.4.6.tgz，没有 bin ）。

* 也可以直接用 Git ，打开Git Bash，然后 clone 最新的源码，具体操作如上。只是网络比较差的话下载不了，因为是多个开发版本的跟踪，所以会占用空间比较大。在spark的目录下运行Git Bash。

  ```shell
  $ git tag # 可以看到有很多版本
  ```

  使用check out 命令回退版本到“发布的”版本，比如v2.4.6。

  ```shell
  $ git checkout v3.0.0
  ```

  <img src="spark_build.assets\image-20200801172903446.png" width="70%">

  



### 设置内存使用

官网中这样写：

```
export MAVEN_OPTS="-Xmx2g -XX:ReservedCodeCacheSize=1g"
```

在Windows系统中如上面所说的设置环境变量，在账户中新建一个环境变量 MAVEN_OPTS：

<img src="spark_build.assets\image-20200801115405672.png" width="70%">



### 编译源码

在源码目录下打开 Git Bash，就是右键，点击“Git Bash Here”，然后输入如下：

```shell
./build/mvn -DskipTests clean package
```

在这个过程中有很多 [Warn] 警告，不要理他。

Tips：clean表示删除之前编译的文件。网络不好时会卡住甚至出错，按住 `ctrl + C `键强制退出，然后输入上面的命令：

```shell
./build/mvn -DskipTests clean package
```

之后再卡住的话，重复上面操作。

__指定 Hadoop 版本__

官方教程：

```shell
# Apache Hadoop 2.6.X
./build/mvn -Pyarn -DskipTests clean package

# Apache Hadoop 2.7.X and later
./build/mvn -Pyarn -Phadoop-2.7 -Dhadoop.version=2.7.3 -DskipTests clean package
```

我这里指定 hadoop2.10.0：

```shell
./build/mvn -Pyarn -Phadoop-2.10 -Dhadoop.version=2.10.0 -DskipTests clean package
```



__编译成功的结果__

编译过程中，程序将编译后的结果放在每个子目录下的 `target` 目录，最后的结果：

<img src="images\image-20200805200300719.png" width="75%">



### 额外说明

* 编译 spark 需要用到 scala 和 zinc，但执行过程中会直接下载好合适的版本，即使你已经安装了scala，它还是会下载合适的版本。

* 事实上可以不安装Maven，它还是会自动帮你安装一个合适的Maven，但是你安装之后，它就使用的所安装的。



### 简单执行

可以在./bin/中点击 spark-shell.cmd，就打开了一个cmd窗口（或者在cmd中执行 ./bin/spark-shell）。可以看到出现如下 Error：

```shell
ERROR Shell: Failed to locate the winutils binary in the hadoop binary path
```

这是因为hadoop在windows下需要`winutils.exe`，即使安装配置了hadoop，也有这样的错误，因为默认情况下hadoop没有`winutils.exe`。

可以在网上找一个，或者在windows上编译hadoop。比如在 https://github.com/steveloughran/winutils 里面下载最相近hadoop版本的  `winutils.exe` 和 `hadoop.dll` 文件，放在hadoop的bin目录下。重新打开spark-shell，此时就没有Error 了。

Tips：安装配置 Hadoop跟安装Maven一样，下载压缩包解压后，设置好  `HADOOP_HOME` 环境变量为对应的安装目录，`path`总增加hadoop的bin目录路径即可。



不管有无Error，都在下面出现了 scala （一门语言）的命令行输入框，可以输入：

```scala
val a = 10;
```

其中 "val" 是指常量定义。具体可以查看scala教程 https://www.runoob.com/scala/scala-tutorial.html 。



## 阅读源码

到这里已经完成了源码的编译。

在 IDEA 中点击 File --> Open （新打开界面就直接 Open...），然后找到 spark 源码下的 pom.xml，点击作为一个project即可。然后慢慢等待 IDEA 解析代码。结果如下图：

<img src="images\image-20200801122550873.png">





__参考资料：__

1. 使用git管理spark和阅读源码 https://linbojin.github.io/2016/01/09/Reading-Spark-Souce-Code-in-IntelliJ-IDEA/
2. 知乎博客 https://zhuanlan.zhihu.com/p/30333691

2. spark 官网编译教程 http://spark.apache.org/docs/latest/building-spark.html