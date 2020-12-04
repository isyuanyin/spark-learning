# sparkPi 阅读

源码位置：spark-3.0.0\examples\src\main\scala\org\apache\spark\examples\SparkPi.scala



## 测试运行

在主目录下打开命令行，执行：

```powershell
.\bin\run-example SparkPi 10 > pi.txt
```

中间有一段日志输出，最后得到结果出现在 pi.txt 文件中。

这是 run-example.sh 的代码：

```shell
if [ -z "${SPARK_HOME}" ]; then
  source "$(dirname "$0")"/find-spark-home
fi

export _SPARK_CMD_USAGE="Usage: ./bin/run-example [options] example-class [example args]"
exec "${SPARK_HOME}"/bin/spark-submit run-example "$@"

```

就是调用 spark-submit 中的某部分

可以用 spark-submit 的方式直接提交它：

```powershell
.\bin\spark-submit --class org.apache.spark.examples.SparkPi --master local[8] .\examples\target\scala-2.12\jars\spark-examples_2.12-3.0.0.jar  100 > pi2.txt
```

执行结果于上面一样



## SparkPi 代码



```java
object SparkPi {
  def main(args: Array[String]): Unit = {
    val spark = SparkSession                                  // spark session 定义
      .builder
      .appName("Spark Pi")
      .getOrCreate()
    val slices = if (args.length > 0) args(0).toInt else 2
    val n = math.min(100000L * slices, Int.MaxValue).toInt // avoid overflow
    val count = spark.sparkContext.parallelize(1 until n, slices).map { i =>
      val x = random * 2 - 1
      val y = random * 2 - 1
      if (x*x + y*y <= 1) 1 else 0
    }.reduce(_ + _)
    println(s"Pi is roughly ${4.0 * count / (n - 1)}")
    spark.stop()
  }
}
```

程序采用蒙特卡洛方法，直接计算单位圆面积，从而得到 pi 的值。
