# 一文搞定JVM调优（原创干货）

## JVM内存模型

![jvm模型](/imgs/jvm/jvm模型.png)

+ 程序计数器：线程私有，记录当前线程字节码执行的位置，占用内存很小，java虚拟机规范中唯一没有规定OOM的区域。
+ java虚拟机栈：线程私有，每个java方法调用时候创建一个栈帧入栈，结束后出栈，所有栈帧出栈，线程结束。
  + 局部变量表：存放方法参数与内部定义的局部变量，编译期就确定最大容量。
  + 操作数栈：方法执行就是各种字节码指令向操作数栈中写入和读取数据，也就是入栈和出栈,编译期就确定栈的最大深度。
  + 动态链接：保存指向运行时常量池中该栈帧所属方法的引用，运行时期间将符合引用转化为直接引用，找到对应的类和方法。在编译期间就能转换为直接引用的叫做静态解析。此处不明白可以先往下看。
  + 方法出口：方法执行完后返回的位置，一般可以是上一个栈帧的计数，如果方法异常，返回位置通过异常处理器来定位。
+ 本地方法栈：同java虚拟机栈类似，执行的是native方法，非java实现，一般由C/C++实现。
+ 堆
  + new出来的对象放在堆中，栈中的局部变量引用指向此处
  + 字符串常量池：存放String字符串的实例
  + 运行时常量池：每个class独有，当类加载到内存中，jvm就会将class常量池中的内容放到运行时常量池中，类在解析之后，将符号引用替换成直接引用。运行时常量区中的字符串常量和字符串常量池保持一致。。
+ 元空间：元空间存储class的信息，堆中的每个对象都会有一个指向元空间中具体实现类的指针。
  + 类描述信息：类的版本、字段、方法、接口等描述信息
  + class常量池：存放编译器生成的各种字面量（常量）和符号引用（一组符号来描述所引用的目标，一般包括类和接口的全限定名、字段的名称和描述符以及方法的名称和描述符，比如People中引用的Name，编译期间不知道Name的直接地址，所以存放的是符号引用，在运行时常量区中会转换成直接引用）
+ 直接内存：不是JVM定义的内存区域，堆外内存。

## 常用JVM参数

+ -Xms ：程序启动时占用内存大小，初始堆大小

+ -Xmx ：程序运行期间最大可占用的内存大小，最大堆大小，超出会抛出OutOfMemory异常。

+ -Xmn ：设置年轻代大小，注意，增大年轻代会减少老年代大小，官方推荐为堆的3/8

+ -Xss   ：设置每个线程的堆栈大小，jdk1.5后为1M

+ -XX:+HeapDumpOnOutOfMemoryError ：JVM在发生内存溢出时自动的生成堆内存快照

+ -XX:HeapDumpPath=<path> ： 堆内存快照会保存在JVM的启动目录下名为java_pid<pid>.hprof 的文件里（在这里<pid>就是JVM进程的进程号），可以通过该命令指定

+ -XX:OnOutOfMemoryError ：内存溢出时，可以指定运行脚本等。例如-XX:OnOutOfMemoryError ="sh cleanup.sh"

+ -XX:PermSize ： 永久代初始大小，jdk1.8后已经移除永久代

+ -XX:MaxPermSize ：最大永久代大小，jdk1.8后已经移除永久代

+ -XX:MetaspaceSize ：首次使用不够而触发FGC的阈值，达到会进行fullgc，后会计算新的FGC值，和此项无关，默认为20M左右。

+ XX:MaxMetaspaceSize ：用于设置metaspace区域的最大值，元空间不使用jvm内存，使用的是本地内存，不设置默认-1无穷大，会根据本地内存大小相关，超过会OOM

+ -XX:+PrintGCDetails：打印GC详细的日志信息

+ -XX:+PrintCommandLineFlags：JVM打印出那些已经被用户或者JVM设置过的详细的XX参数的名称和值

+ -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=1506：开启远程调试

  -XDebug：表示虚拟机启用调试功能

  -Xrunjdwp：加载JDWP

  transport：调试程序JVM使用的进程之间通讯方式

  dt_socket：socket通讯

  server=y/n：JVM是否需要作为调试服务器执行

  address：调试服务器监听的端口号

  suspend=y/n：调试客户端建立连接之后启动虚拟机

+ -XX:+UseSerialGC：虚拟机运行在Client模式下的默认值，Serial+Serial Old

+ -XX:+UseParNewGC：ParNew+Serial Old，在JDK1.8被废弃，在JDK1.7还可以使用

+ XX:+UseConcMarkSweepGC：ParNew+CMS+Serial Old

+ -XX:+UseParallelGC：虚拟机运行在Server模式下的默认值，Parallel Scavenge+Serial Old(PS Mark Sweep)

+ -XX:+UseParallelOldGC：Parallel Scavenge+Parallel Old，常用

+ -XX:+UseG1GC：G1+G1，jdk1.8后可用

## JVM调优实战

### 1. java.lang.OutOfMemoryError: Java heap space

原因：在JVM中如果98％的时间是用于GC且可用的 Heap size 不足2％的时候将抛出此异常信息。

实例：写一个无限循环并设置jvm参数如下：

```java
JVM参数: -Xms1m -Xmx1m -XX:+HeapDumpOnOutOfMemoryError 

import java.util.ArrayList;
import java.util.List;

public class Test {
    public static void main(String[] args) {
        List<Test> list = new ArrayList<Test>();
        while(true) {
            list.add("test");
        }
    }
}
```

执行后会在对应运行目录下生成dump快照文件，例如:java_pid33176.hprof

此时win+R ==>  cmd ==>  jvisualvm ==> 文件 ==> 装入该文件

![dump1](/imgs/jvm/dump1.png)

![dump2](/imgs/jvm/dump2.jpg)

可以看到此时main线程错误，String实例和占用内存最多，如果是对象实例过多，此处会显示对应的对象。

### 

