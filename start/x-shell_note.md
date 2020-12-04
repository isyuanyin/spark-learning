## 😉 Shell 笔记

### 开始示例

```shell
#!/bin/bash
#This is to show what a example looks like.
echo "Our first example"
echo # This inserts an empty line in output.
echo "We are currently in the following directory"
pwd
echo
echo "This ddirectory contains the following files"
ls
```

注释：

#! /bin/bash   #!跟shell 命令的完全路径。作用：显示后期命令以哪种shell来执行这些命令。如不指定shell，以当前shell作为执行的shell。

#This is to show what a example looks like.  shell中以#开头表示注释。

shell程序一般以.sh结尾



总结：

__创建shell程序的步骤：__

* 第一步：创建一个包含命令和控制结构的shell文件。

* 第二步：修改这个文件的权限使它可以执行。

  使用chmod u+x filename

* 第三步：执行

  方法1：./example.sh

  方法2：使用绝对路径

  方法3：bash example.sh



__👀Tips ：修改文件权限__

修改文件权限使用 chmod 指令（change mode）。该指令常用的有两种使用方式：

* __chmod   abc   filename__

指令中的a、b、c分别表示一个数字，其中a对应文件所有者权限，b对应文件所有者所在组权限，c对应其他身份权限。

对于a、b、c各自来讲，它们都是0~7的数字，对应r、w、x三个二进制位按序组成的二进制数，举个例子，如果是只可读，对应的二进制数就是“100”，也就是4。

如果要将上述client.cpp文件权限改为 “文件所有者可读可写可执行（rwx），其余身份只可读（r--）”，那么就可以使用如下指令：

```shell
chmod 744 client.cpp
```

* __chmod   u/g/o/a   +/-   r/w/x   filename__

该指令除了 chmod 和 filename 之外，还有三个部分：

1️⃣ 描述文件权限身份。u表示文件所有者、g表示文件所有者所在组、o表示其他用户、a表示三者全部。可以搭配使用，如ug表示文件所有者及其所在组；

2️⃣ 指定权限配置行为。`+` 表示添加权限，`-`表示删除权限；

3️⃣ 权限类型。分别对于可读可写可执行（rwx）。

例如，通过chmod的第一种方式，我已经将client.cpp的权限改为“文件所有者可读可写可执行，其余身份只可读”，如果我现在想删除文件所有者的可执行权限(u -x)，增加文件所有者所在组和其他身份的可写和可执行权限(go +wx)，就可以使用如下指令：

```shell
chmod u-x,go+wx client.cpp
```



### shell变量

shell 两类变量：临时变量和永久变量

* 临时变量：是shell程序内部定义的变量，其使用范围仅限于定义它的程序，对其它程序不可见。

* 永久变量：是环境变量，其值不随shell脚本的执行结束而消失。例如：$PATH用作运行某个命令的时候，本地查找不到某个命令或文件，会到这个声明的目录中查找。

__用户自定义变量：__由字母或下划线开始，由字母、数字或下划线组成；并且不区分大小写；长度没有限制。__使用变量值时__，要在变量名前加前缀 __`$`__，例如 `$A` 或者 `${A}`

__变量赋值：__赋值号 `=` 左右没有空格

```shell
# 示例1
A=aaa
# 示例2：赋值date命令的结果
A='date'
# 示例3
B=$(ls -l)
# 示例4
A=$B
echo $A
```

__列出所有变量：__

```shell
set
set | grep A
```

给变量赋值多个单词：

```shell
NAME="Mike Ron"
```

Tips：字符串中有空格需要用引号括起来。

__单引号和双引号的区别：__

* 单引号之间的内容原封不动地指定给了变量。

* 双引号取消了空格的作用，特殊符号的含义保留。

__删除变量：__

```shell
unset A
```

__位置变量和特殊变量：__

__位置变量：__Shell 解释执行用户的命令时，将命令行的第一个字作为命令名，而其它作为参数。由出现在命令行上的位置确定的参数称为位置参数。使用 `$N` 来表示。

示例：

```shell
./example.sh file1 file2 file3
```

\$0 表示这个程序的文件名 example.sh

\$n 表示这个程序的第n个参数值



__特殊变量：__

有些变量是一开始执行Script脚本时就会设定，且不能被修改，但我们不叫它只读的系统变量，而叫它特殊变量。这些变量当一执行程序时就有了，以下是一些特殊变量：

```shell
$*  # 这个程序的所有参数
$@  # 所有参数构成的数组
$#  # 这个程序的参数个数
$$  # 这个程序的PID
$!  # 执行上一个后台程序的PID
$?  # 执行上一个指令的返回值
$0  # xxx.sh，第0个参数
$1  # 第一个参数
```



### 变量表达式

__read 命令__

作用：从键盘读入数据，赋给变量

```shell
read a b c
1 32 2
echo $a $b $c #返回1 32 2
```



__expr 命令__

作用：shell变量的算数运算，整数运算

```shell
expr 3 + 5
expr $var1 \* $var2
```



__变量测试语句：__

__test 命令__

格式：test 测试条件

测试范围：整数，字符串，文件

字符串和变量：

```shell
test str1==str2 是否相等
test str1 测试字符串是否不空
test -n str1 测试字符串是否为空
test -z str1测试字符串是否为空
```

测试整数：

```shell
test int1 -eq int2
test int1 -ge int2
test int1 -gt int2
test int1 -le int2

# 说明：可以省略test，写成: [int1 -lt int2]
```

文件测试：

```shell
test -d file # 测试是否为目录
test -f file
test -x file
test -r file
test -w file
test -e file 测试文件是否存在
test -s file 测试大小是否为空

# 说明
test -x file 简写成：[-x file]
```



### 流程控制

if语句

语法：

```shell
if 条件 
then 
	语句
fi

# 或者
if 条件; then
	语句
fi
```

`;`分号：表示两个命令写在一行。



语法：

```shell
if 条件
then
	语句
else
	语句
fi
```



多个条件的联合

* -a 或 && : 逻辑与。

* -o 或 ||：逻辑或。



更复杂的if语句

语法

```shell
if 条件1 ; then
	命令1
elif 条件2 ; then
	命令2
elif 条件3; then
	命令3
else
	命令4
fi
```



综合实例：测试文件类型

```shell
#! /bin/bash
echo "input a file name"
read file_name
if [ -d $file_name ] ; then
	echo " $file_name is a dir"
elif [ -f $file_name] ; then
	echo " $file_name is a file"
elif [ -c $file_name -o -b $file_name ] ; then
	echo " $file_name is a device file"
else
	echo " $file_name is an unknow file"
fi

```



case语句

适用于多分支

格式：

```shell
case 变量 in
字符串1) 命令列表1
;;
...
字符串n)命令列表n
;;
esac   #esac是case倒着写
```





### 循环控制

__for...done语句__

格式：

```shell
for 变量 in 名字表
do
	命令列表
done
```



__while循环语句__

格式：

```shell
while 条件
do 
	命令列表
done
```



__使用(())扩展__shell中算数运算的使用方法

使用`[]`时候，必须保证运算符与算数之间有空格。四则运算也只能借助：expr命令完成。这里的双括号`(())`结构语句，就是对shell中算数及赋值运算的扩展。

使用方法：

```
((表达式1， 表达式2))
```

特点：

* 在双括号结构中，所有表达式可以像C语言一样，如：a++，b--等。
* 在双括号结构中，所有变量可以不加入： `$` 符号前缀
* 双括号可以进行逻辑运算，四则运算
* 双括号结构扩展了for，while，if条件测试运算
* 支持多个表达式运算，各个表达式之间用逗号 `,` 分开

示例：

```shell
#！/bin/bash
echo "The while loop example"
echo
VAR1=1
while ((VAR1<100))
do
	echo "Value of the variabble is : $VAR1"
	((VAR1=VAR1*2))
done
echo
echo "The loop execution is finished"
```



__跳出循环：__break和continue



### 注意事项

输入不换行：

```shell
echo -n "Please Enter the line number: "
read Line
```

或者

```shell
read -p "Please Enter the line number: " Line
```

输出`*`符号：

```shell
echo "*"
而不是
echo * # 这里*会匹配当前目录下所有文件名
```



使用cat

```shell
cat <<EOF
```



__将windows中的脚本导入到Linux系统执行报错__

因为windows和linux对换行的解释不同

```shell
rpm -ivh /mnt/Packages/dos2unix-3.1-37.el6.x86_64.rpm
# 安装dos2unix
dos2unix filename.sh
```



### shift：参数左移指令

每执行以此，参数序列顺次左移一个位置，$#的值减1，用于分别处理每个参数，移出去的参数，不可再用。

```shell
sum = `expr $sum + $1`
shift
```



### shell函数

函数的定义：

```shell
函数名()
{
	命令序列
}
# 或者：
function 函数名() # function 可以不写
{
	命令序列
}
```

注意：函数调用时，不带`()`

调用语法：

```shell
函数名 参数1 参数2 ...
```

函数中的变量均为全局变量，没有局部变量

调用函数时，可以传递参数。在函数中使用\$1、\$2 ...来引用传递的参数



### 正则表达式

使用 grep

```shell
grep [-v][-n] [搜索样式] [文件或目录]
-v 不匹配
-n 显示行号
```



1.正则表达式中特殊字符

(1) ^word：待搜索的字符串(word)在行首！

(2) word$：将行尾为word的那一行打印出来

(3) \：将特殊符号的特殊意义去除

(4) *：重复零个到无穷多个的前一个字符

(5) [list]：字符集合，里面列出想要选择的字符

(6) 搜寻不以#号开头的行：^\[^#] 

(7) [n1-n2]：n1到n2的字符集

(8) . ：表示一个任意字符