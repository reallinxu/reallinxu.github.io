---
title: "哇，又又又又OOM了"
tags:
  - java
id: 1001
categories:
  - java
date: 2020-9-20 10:00:00
---

## 哇，又又又又OOM了！！！

没错，我也学了一下标题党，别骂我，我很怂 Orz...

OOM是每个码农都会害怕的玩意，那导致OOM的场景有哪些呢？本文主要模拟OOM的场景，如果有帮助请来个三连，关注在看再点赞，3Q everybody

### 堆内存溢出

堆内存溢出应该是最常见的一种了，当看到那熟悉的异常信息时，是不是都是虎躯一震，背后发凉呢？

```java
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:3210)
	at java.util.Arrays.copyOf(Arrays.java:3181)
	at java.util.ArrayList.grow(ArrayList.java:261)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:235)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:227)
	at java.util.ArrayList.add(ArrayList.java:458)
	at com.reallinxu.main.HeapOOM.main(HeapOOM.java:15)
```

难搞哦，这可咋办哦？莫慌，下面模拟一下产生的场景，并助你解决之。

首先，我们要写一个辣鸡代码，没错，不辣鸡怎么能OOM呢，下面看代码:

```java
import java.util.ArrayList;
import java.util.List;

/**
 * 设置最大堆为2M,模拟堆内存溢出
 * vm options: -Xms2M -Xmx2M -XX:+HeapDumpOnOutOfMemoryError
 */
public class HeapOOM {

    public static void main(String[] args) {
        List<HeapOOM> list = new ArrayList<>();
        while (true) {
            list.add(new HeapOOM());
        }
    }
}
```

辣鸡代码的好帮手就是无限循环了，启动时加上vm参数，当OOM时候会自动生成hprof文件，可以通过jdk自带的VisualVm打开进行分析，直接命令执行jvisualvm或者找到jdk安装目录下bin里面的jvisualvm.exe文件，装入生成的hprof，然后就开始胡乱分析吧~

![实例1](/imgs/oom.jpg)

不看不知道，一看吓一跳，我滴龟龟，HeapOOM这个实例咋占用这么多哎，赶紧搜代码这玩意在哪new了，哦，我就三行代码，行吧找到了，干掉无限循环就完了，成功解决。当然，生产的不会这样简单，还需要根据场景自己发挥去吧。

### 元空间内存溢出

jdk1.8之后，永久代被永久的干掉了，你再也看不到java.lang.OutOfMemoryError: PermGen error的异常了，代替的是元空间，元空间存储class的信息，堆中的每个对象都会有一个指向元空间中具体实现类的指针。正常情况未指定元空间大小时，元空间的大小和机器内存有关。

随便写个main方法，启动时加上参数-XX:MetaspaceSize=2m -XX:MaxMetaspaceSize=2m，你可以看到下面很少看到的OOM，请珍惜你的这次机会，错过可能就是一辈子。

```java
Error occurred during initialization of VM
OutOfMemoryError: Metaspace
```

这个咋解决呢，怎么调小的，怎么调大就完了呗，一般生产环境根据内存指定好最大元空间大小，基本上是不会出现这个问题滴。

### 栈内存溢出

栈溢出一般是方法递归太深或者陷入死循环，或者局部变量太大导致的溢出，下面直接上代码模拟。

```java
/**
 * 设置栈大小，模拟栈溢出
 * vm options: -Xss160k
 */
public class StackOOM {

    public static void test() {
        test();//死循环
    }

    public static void main(String[] args) {
        Thread th = new Thread(() -> test());
        th.start();
        //模拟主线程不结束
        while (true){}
    }
}
```

当然又用到了死循环（毕竟我也不会用其他的），运行时增加参数设置栈大小，默认每个线程栈大小为1M，眨眼间栈就爆了，但是主线程不会受影响结束，开心不？

```java
Exception in thread "Thread-0" java.lang.StackOverflowError
	at com.reallinxu.main.StackOOM.test(StackOOM.java:10)
	at com.reallinxu.main.StackOOM.test(StackOOM.java:10)
	at com.reallinxu.main.StackOOM.test(StackOOM.java:10)
	at com.reallinxu.main.StackOOM.test(StackOOM.java:10)
```

线程栈大小分配的越大，占用内存越多，能容纳的线程就越少，反之，就会出现栈爆炸了，一般默认1M已经完全够用了。

### 直接内存溢出

直接内存溢出遇到的也是比较少的，直接内存也叫堆外内存，是不受JVM管理的，所以dump出来的快照也是看不到的，只能通过top命令查看当前内存占用，申请后需要手动释放，此处我们用nio的ByteBuffer进行模拟。

```java
import java.nio.ByteBuffer;

/**
 * 通过NIO获取直接内存，设置直接内存大小为2M，不设置默认为最大堆大小
 * vm options: -XX:MaxDirectMemorySize=2M
 */
public class DirectOOM {
    public static void main(String[] args) {
        ByteBuffer.allocateDirect(2 * 1024 * 1024 + 1);
    }
}
```

设置直接内存最大为2M，然后我们来申请2M+1（我可真的是一个小机灵鬼）

```java
Exception in thread "main" java.lang.OutOfMemoryError: Direct buffer memory
	at java.nio.Bits.reserveMemory(Bits.java:693)
	at java.nio.DirectByteBuffer.<init>(DirectByteBuffer.java:123)
	at java.nio.ByteBuffer.allocateDirect(ByteBuffer.java:311)
	at com.reallinxu.main.DirectOOM.main(DirectOOM.java:11)
```

直接内存也可以用作堆外缓存，Ehcache包就利用了直接内存，使用直接内存可以减少GC的压力，从而减少GC对业务的影响。

