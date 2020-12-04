# 远程连接：SSH相关工具

本文将介绍ssh远程连接方式，免密登录方式和文件传输工具。



## ssh 远程连接

示例：

```shell
ssh yuanyin@192.168.13.2
```

然后输入主机密码即可。



## ssh 免密登录

分为两个步骤，第一步生成RSA密钥，第二步是将公钥发送至远程服务器

### 生成RSA密钥

在主机端输入如下：

```
ssh-keygen -t rsa
```

生成一个RSA密码机制的公钥与私钥（关于RSA加密机制可以参考相关资料），分别对应着文件 `id_rsa.pub` 和 `id_rsa` 。

生成过程：

```shel
yuanyin@ubuntu$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/yuanyin/.ssh/id_rsa): 
Created directory '/home/yuanyin/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/usera/.ssh/id_rsa.
Your public key has been saved in /home/usera/.ssh/id_rsa.pub.
The key fingerprint is:
39:f2:fc:70:ef:e9:bd:05:40:6e:64:b0:99:56:6e:01 yuanyin@serverA
The key's randomart image is:
+--[ RSA 2048]----+
|          Eo*    |
|           @ .   |
|          = *    |
|         o o .   |
|      . S     .  |
|       + .     . |
|        + .     .|
|         + . o . |
|          .o= o. |
+-----------------+
```

在生成过程中询问2个问题：

* 保存目录。默认是家目录下的文件夹` /home/yuanyin/.ssh/` ，在linux是隐藏的；Windows下的家目录是`C:\Users\yuanyin\` ，密钥保存在`C:\Users\yuanyin\.ssh` 中。
* passphrase 通行句子，一般直接回车不填写；在确认询问中输入同样的句子

### 发送公钥并保存

然后将公钥文件`id_rsa.pub`发送给服务器端。可以使用scp指令完成，具体可看下一节的scp文件传输，这里有一个示例。

```shell
 scp ./id_rsa.pub yuanyin@192.168.13.2:~/.ssh
```

然后将公钥加入授权文件 `authorized_keys` 中（如果没有授权文件，就生成一个）

```shell
touch authorized_keys  # 没有该文件的情况下执行
cat id_rsa.pub >> authorized_keys
```



## scp 文件传输

示例：

```shell
 scp -P 2233 filename yuanyin@192.168.13.2:/usr/local/src
```

参数：

> -P [port]: 可以通过-P指点端口

