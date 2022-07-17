# 第一章 进程与线程

**本章内容**

- 进程和线程的概念
- 并行和并发的概念
- 线程基本应用



## 1.1 进程与线程

**进程**

- 进程就是用来加载指令、管理内存、管理IO的，它是作为资源分配的最小单位
- 在Windows中的进程是不活动，只是作为线程的容器
- 进程可以看成程序的一个实例，当程序被执行时，就会从磁盘中加载该程序的代码到内存中，开启一个进程。统一程序可以开多个实例进程，如（QQ）也可以只能开一个，如（qq音乐）

**线程**

- 一个线程就是一个指令流，将指令流中的指令以一定的顺序交给CPU执行
- 线程是最小的调度单位
- 一个进程内可以有多个线程



进程VS线程

- 进程基本上相互独立的，而线程存在于进程内，是进程的一个子集 
- 进程拥有共享的资源，如内存空间等，供其内部的线程共享 
- 进程间通信较为复杂

  - 同一台计算机的进程通信称为 IPC(Inter-process communication) 

  - 不同计算机之间的进程通信，需要通过网络，并遵守共同的协议，例如 HTTP

- 线程通信相对简单，因为它们共享进程内的内存，一个例子是多个线程可以访问同一个共享变量 
- 线程上下文切换成本一般上要比进程上下文切换低



## 1.2 并行与并发

**概念**

- 并发(concurrent)是同一时间应对(dealing with)多件事情的能力

- 并行(parallel)是同一时间动手做(doing)多件事情的能力



## 1.3 同步与异步

以**调用方角度**来讲，如果

- 需要等待结果返回，才能继续运行就是同步 

- 不需要等待结果返回，就能继续运行就是异步



# 第二章 Java线程

**本章内容**

- 创建和运行线程 

- 查看线程
- 线程 API 

- 线程状态



## 2.1 创建和运行线程

### 2.1.1 直接使用Thread

```java
// 创建线程对象
Thread t = new Thread() {
		public void run() { 
      	// 要执行的任务
    } 
};
// 启动线程 t.start();
```

例如

```java
// 构造方法的参数是给线程指定名字，推荐 
Thread t1 = new Thread("t1") {
		@Override
		// run 方法内实现了要执行的任务 
  	public void run() {
				log.debug("hello"); 
    }
}; 
t1.start();
```

输出

```
19:19:00 [t1] c.ThreadStarter - hello
```



### 2.1.2 Runnable配合Thread

把【线程】和【任务】(要执行的代码)分开

- Thread 代表线程
- Runnable 可运行的任务(线程要执行的代码)

```java
Runnable runnable = new Runnable() { 
   public void run(){
			// 要执行的任务 
   }
};
// 创建线程对象
Thread t = new Thread( runnable ); // 启动线程
t.start();
```

例如

```java
// 创建任务对象
Runnable task2 = new Runnable() {
		@Override
		public void run() { 
      log.debug("hello");
		} 
};
// 参数1 是任务对象; 参数2 是线程名字，推荐 
Thread t2 = new Thread(task2, "t2"); 
t2.start();
```

输出

```
19:19:00 [t2] c.ThreadStarter - hello
```



Java 8 以后可以使用 lambda 精简代码

```java
// 创建任务对象
Runnable task2 = () -> log.debug("hello");
// 参数1 是任务对象; 参数2 是线程名字，推荐 
Thread t2 = new Thread(task2, "t2"); 
t2.start();
```

**\* 原理之 Thread 与 Runnable 的关系**

分析 Thread 的源码，理清它与 Runnable 的关系 

- 第一种，采用匿名内部类的方式，创建了Thread子类的对象，并重写了Thread中的Run（）

- 第二种，将一个Runnable对象传给Thread对象中的构造方法，调用Thread对象中的Run（），判断Runnable对象是否为空，不为空就调用Runnable对象的Run（）

**小结**

- 方法1是把线程和任务合并在了一起，方法2是把线程和任务分开了
- 用 Runnable 更容易与线程池等高级 API 配合
- 用 Runnable 让任务类脱离了 Thread 继承体系，更灵活



### 2.1.3 FutureTask配合Thread

FutureTask 能够接收 Callable 类型的参数，用来处理有返回结果的情况

```java
// 创建任务对象
FutureTask<Integer> task3 = new FutureTask<>(() -> {
		log.debug("hello");
		return 100; 
});
// 参数1 是任务对象; 参数2 是线程名字，推荐 
new Thread(task3, "t3").start();
// 主线程阻塞，同步等待 task 执行完毕的结果 
Integer result = task3.get(); 
log.debug("结果是:{}", result);
```

输出

```
 19:22:27 [t3] c.ThreadStarter - hello 
 19:22:27 [main] c.ThreadStarter - 结果是:100
```



## 2.2 查看进程线程的方法

#### 2.2.1 windows

```
任务管理器可以查看进程和线程数，也可以用来杀死进程
tasklist 查看进程 
taskkill 杀死进程
```



#### 2.2.2 linux

```
ps -fe 查看所有进程
ps -fT -p <PID> 查看某个进程(PID)的所有线程 kill 杀死进程
top 按大写 H 切换是否显示线程
top -H -p <PID> 查看某个进程(PID)的所有线程
```



#### 2.2.3 Java

```
jps 命令查看所有 Java 进程
jstack <PID> 查看某个 Java 进程(PID)的所有线程状态 
jconsole 来查看某个 Java 进程中线程的运行情况(图形界面)
```



**jconsole远程监控配置**

- 需要以如下方式运行你的 java 类

  ```
  java -Djava.rmi.server.hostname=`ip地址` -Dcom.sun.management.jmxremote - Dcom.sun.management.jmxremote.port=`连接端口` -Dcom.sun.management.jmxremote.ssl=是否安全连接 - Dcom.sun.management.jmxremote.authenticate=是否认证 java类
  ```

- 修改 /etc/hosts 文件将 127.0.0.1 映射至主机名 	


如果要认证访问，还需要做如下步骤

- 复制 jmxremote.password 文件
- 修改 jmxremote.password 和 jmxremote.access 文件的权限为 600 即文件所有者可读写 
- 连接时填入 controlRole(用户名)，R&D(密码)



## 2.3  原理之线程运行

#### 2.3.1 栈与栈帧

每个线程启动后，Java 虚拟机就会为其分配一块栈内存

- 每个栈由多个栈帧(Frame)组成，对应着每次方法调用时所占用的内存 
- 每个线程只能有一个活动栈帧，对应着当前正在执行的那个方法



#### 2.3.2 线程上下文切换 

Thread Context Switch，因为以下一些原因导致 cpu 不再执行当前的线程，转而执行另一个线程的代码

- 线程的 cpu 时间片用完
- 垃圾回收
- 有更高优先级的线程需要运行
- 线程自己调用了 sleep、yield、wait、join、park、synchronized、lock 等方法



当 Context Switch 发生时，需要由操作系统保存当前线程的**状态**，并恢复另一个线程的状态，Java 中对应的概念就是程序计数器(Program Counter Register)，它的作用是记住下一条 jvm 指令的执行地址，是线程私有的

- 状态包括程序计数器、虚拟机栈中每个栈帧的信息，如局部变量、操作数栈、返回地址等 

- Context Switch 频繁发生会影响性能



## 2.4 常见方法

### 2.4.1 概述

|      方法名      | static | 功能说明                                                     | 注意                                                         |
| :--------------: | :----: | :----------------------------------------------------------- | :----------------------------------------------------------- |
|     start()      |        | 启动一个新线程，在新的线程运行 run 方法中的代码              | start 方法只是让线程进入就绪，里面代码不一定立刻 运行(CPU 的时间片还没分给它)。每个线程对象的 start方法只能调用一次，如果调用了多次会出现 IllegalThreadStateException |
|      run()       |        | 新线程启动后会调用的方法                                     | 如果在构造 Thread 对象时传递了 Runnable 参数，则 新线程启动后会调用 Runnable 中的 run 方法，否则默认不执行任何操作，但可以创建 Thread 的子类对象，来覆盖默认行为 |
|      join()      |        | 等待线程运行结束                                             |                                                              |
|   join(long n)   |        | 等待线程运行结束,最多等待 n 毫秒                             |                                                              |
|     getId()      |        | 获取线程长整型的 id                                          | id唯一                                                       |
|    getName()     |        | 获取线程名                                                   |                                                              |
| setName(String)  |        | 修改线程名                                                   |                                                              |
|  getPriority()   |        | 获取线程优先级                                               |                                                              |
| setPriority(int) |        | 修改线程优先级                                               | java中规定线程优先级是1~10 的整数，如果 cpu 比较忙，较大的优先级能提高该线程被 CPU 调度的机率，但它仅仅是一个提示，调度器可以忽略它，比如cpu 闲时，优先级几乎没作用 |
|    getState()    |        | 获取线程状态                                                 | Java 中线程状态是用 6 个 enum 表示，分别为: NEW, RUNNABLE, BLOCKED, WAITING,TIMED_WAITING, TERMINATED |
| isInterrupted()  |        | 判断线程是否被打断                                           | 不会清除打断标记                                             |
|    isAlive()     |        | 判断线程是否存活（没有运行完毕）                             |                                                              |
|   interrupt()    |        | 打断线程                                                     | 如果被打断线程正在 sleep，wait，join 会导致被打断的线程抛出 InterruptedException，并清除打断标记 ;如果打断的正在运行的线程，则会设置打断标记 ;park 的线程被打断，也会设置打断标记 |
|  interrupted()   | static | 判断当前线程是否被打断                                       | 会清除打断标记                                               |
| currentThread()  | static | 获取当前正在执行的线程                                       |                                                              |
|  sleep(long n)   | static | 让当前执行的线 程休眠n毫秒，休眠时让出 cpu 的时间片给其它线程 |                                                              |
|     yield()      | static | 提示线程调度器 让出当前线程对CPU的使用                       | 主要是为了测试和调试                                         |



### 2.4.2 start()与run()

```java
public static void main(String[] args) { 
  	Thread t1 = new Thread("t1") {
				@Override
				public void run() { 
          	log.debug(Thread.currentThread().getName()); 				 
          	FileReader.read(Constants.MP4_FULL_PATH);
				} 
		};
		t1.run(); //方式一
  	t1.start();	//方式二
		log.debug("do other things ..."); 
}
```

**总结**

- 直接调用 run 是在主线程中执行了 run，没有启动新的线程
- 使用 start 是启动新的线程，通过新的线程间接执行 run 中的代码



### 2.4.3 sleep()与yield()

**sleep**

- 调用 sleep 会让当前线程从 Running 进入 Timed_Waiting 状态(阻塞)
- 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException 
- 睡眠结束后的线程未必会立刻得到执行
- 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获得更好的可读性



**yield**

- 调用 yield 会让当前线程从 Running 进入 Runnable 就绪状态，然后调度执行其它线程，如果此时没有其他线程，可能状态转换失败，也就是让出CPU执行时间片失败，继续运行。

- 具体的实现依赖于操作系统的任务调度器



### 2.4.4 join()

```java
static int r1 = 0;
static int r2 = 0;
public static void main(String[] args) throws InterruptedException {
		test3(); 
}
public static void test3() throws InterruptedException { 
  	Thread t1 = new Thread(() -> {
				sleep(1);
				r1 = 10; 
    });
		long start = System.currentTimeMillis(); 
		t1.start();
		// 线程执行结束会导致 join 结束
		t1.join(1500); //有时效的join
  	t1.join(500) // 没等待时间
		long end = System.currentTimeMillis();
		log.debug("r1: {} r2: {} cost: {}", r1, r2, end - start);
}
```

有时效的join

```
20:48:01.320 [main] c.TestJoin - r1: 10 r2: 0 cost: 1010
```

没有等待时间

```
20:52:15.623 [main] c.TestJoin - r1: 0 r2: 0 cost: 502
```

#### 原理

是调用者轮询检查线程 alive 状态

```java
t1.join();

synchronized (t1) {
// 调用者线程进入 t1 的 waitSet 等待, 直到 t1 运行结束 
  while (t1.isAlive()) {
			t1.wait(0); 
  }
}
```



### 2.4.5 interrupt()

#### **打断阻塞**

打断sleep、wait、join的线程，会清空打断标记

```java
private static void test1() throws InterruptedException { 
  	Thread t1 = new Thread(()->{
				sleep(1); 
    }, "t1");
		t1.start();
		sleep(0.5);
		t1.interrupt();
		log.debug(" 打断状态: {}", t1.isInterrupted());
}
```

```
java.lang.InterruptedException: sleep interrupted
		at java.lang.Thread.sleep(Native Method)
		at java.lang.Thread.sleep(Thread.java:340)
		at java.util.concurrent.TimeUnit.sleep(TimeUnit.java:386) 
		at cn.itcast.n2.util.Sleeper.sleep(Sleeper.java:8)
		at cn.itcast.n4.TestInterrupt.lambda$test1$3(TestInterrupt.java:59)
		at java.lang.Thread.run(Thread.java:745) 
		21:18:10.374 [main] c.TestInterrupt - 打断状态: false
```

#### **打断正常**

打断正常运行的线程，会设置打断标记

```java
private static void test2() throws InterruptedException { 
  	Thread t2 = new Thread(()->{
				while(true) {
						Thread current = Thread.currentThread(); 
          	boolean interrupted = current.isInterrupted(); 
          	if(interrupted) {
								log.debug(" 打断状态: {}", interrupted);
								break; 
            }
				}
		}, "t2");
		t2.start();
		sleep(0.5);
		t2.interrupt(); 
}
```

```
20:57:37.964 [t2] c.TestInterrupt - 打断状态: true
```





#### 终止模式-两阶段终止

Two Phase Termination，让一个线程优雅的终止另一个线程，即让另一个线程在终止前做一些善后工作。

**错误思想**

- 使用线程对象的stop()停止线程
  - stop()会真正杀死线程，如果这时线程锁住了共享资源，那么当它被杀死后就再也没有机会释放锁，其他线程将会永远无法获取锁

- 使用System.exit()停止线程
  - 目的仅是停止一个线程，但这种做法会让整个程序都停止



**应用场景**

- 自定义监控线程，能手动停止、启动



**程序流程**

![image-20220716171737002](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207161717337.png)



**程序实现**

```java
@Slf4j(topic = "c.TPTInterrupt")
class TPTInterrupt {
    private Thread thread;

    public void start(){
        thread = new Thread(() -> {
            while(true) {
                Thread current = Thread.currentThread();
                if(current.isInterrupted()) {
                    log.debug("料理后事");
                    break;
                }
                try {
                    Thread.sleep(1000);
                    log.debug("将结果保存");
                } catch (InterruptedException e) {
                    current.interrupt();
                }
            }
        },"监控线程");
        thread.start();
    }

    public void stop() {
        thread.interrupt();
    }
}
```

#### 打断park()

打断park线程，不会清除打断标记

```java
private static void test3() throws InterruptedException { 
  	Thread t1 = new Thread(() -> {
				log.debug("park...");
				LockSupport.park();
				log.debug("unpark...");
				log.debug("打断状态:{}", Thread.currentThread().isInterrupted());
		}, "t1"); 
  	t1.start();
		sleep(0.5);
		t1.interrupt(); 
}
```

```
 21:11:52.795 [t1] c.TestInterrupt - park... 
 21:11:53.295 [t1] c.TestInterrupt - unpark... 
 21:11:53.295 [t1] c.TestInterrupt - 打断状态:true
```

如果打断标记已经是true，则park会失效

```java
private static void test4() { 
  	Thread t1 = new Thread(() -> {
				for (int i = 0; i < 5; i++) {
						log.debug("park...");
						LockSupport.park();
						log.debug("打断状态:{}", Thread.currentThread().isInterrupted());
				} 
    });
		t1.start();
		sleep(1);
		t1.interrupt();
}
```

```
21:13:48.783 [Thread-0] c.TestInterrupt - park... 
21:13:49.809 [Thread-0] c.TestInterrupt - 打断状态:true 
21:13:49.812 [Thread-0] c.TestInterrupt - park... 
21:13:49.813 [Thread-0] c.TestInterrupt - 打断状态:true 
21:13:49.813 [Thread-0] c.TestInterrupt - park... 
21:13:49.813 [Thread-0] c.TestInterrupt - 打断状态:true 
21:13:49.813 [Thread-0] c.TestInterrupt - park... 
21:13:49.813 [Thread-0] c.TestInterrupt - 打断状态:true 
21:13:49.813 [Thread-0] c.TestInterrupt - park... 
21:13:49.813 [Thread-0] c.TestInterrupt - 打断状态:true
```

*可以使用**Thread.interrupted()**清除打断标记

### 2.4.6 不推荐的方法

还有一些不推荐使用的方法，这些方法已过时，容易破坏同步代码块，让锁无法释放，造成线程死锁

| 方法名    | static | 功能说明           |
| --------- | ------ | ------------------ |
| stop()    |        | 停止线程运行       |
| suspend() |        | 挂起(暂停)线程运行 |
| resume()  |        | 恢复线程运行       |



## 2.5 主线程与守护线程

默认情况下，Java 进程需要等待所有线程都运行结束，才会结束。有一种特殊的线程叫做守护线程，只要其它非护线程运行结束了，即使守护线程的代码没有执行完，也会强制结束。

**例如**

```java
log.debug("开始运行..."); 
Thread t1 = new Thread(() -> {
		log.debug("开始运行..."); 
  	sleep(2); 
  	log.debug("运行结束...");
}, "daemon");
// 设置该线程为守护线程 
t1.setDaemon(true); 
t1.start();
sleep(1); 
log.debug("运行结束...");
```

**输出**

```
 08:26:38.123 [main] c.TestDaemon - 开始运行... 
 08:26:38.213 [daemon] c.TestDaemon - 开始运行... 
 08:26:39.215 [main] c.TestDaemon - 运行结束...
```

**注意**

- 垃圾回收器线程就是一种守护线程
- Tomcat 中的 Acceptor 和 Poller 线程都是守护线程，所以 Tomcat 接收到 shutdown 命令后，不会等 待它们处理完当前请求



## 2.6 线程状态

#### 2.6.1 五种状态

这是从**操作系统**层面来描述的

![image-20220716175917204](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207161759409.png)

- 【初始状态】仅是在语言层面创建了线程对象，还未与操作系统线程关联

- 【可运行状态】指该线程已经被创建(与操作系统线程关联)，可以由 CPU 调度执行 
- 【运行状态】指获取了 CPU 时间片运行中的状态
  - 当 CPU 时间片用完，会从【运行状态】转换至【可运行状态】，会导致线程的上下文切换 
- 【阻塞状态】

  - 如果调用了阻塞 API，如 BIO 读写文件，这时该线程实际不会用到 CPU，会导致线程上下文切换，进入 【阻塞状态】


  -  等 BIO 操作完毕，会由操作系统唤醒阻塞的线程，转换至【可运行状态】 
  -  与【可运行状态】的区别是，对【阻塞状态】的线程来说只要它们一直不唤醒，调度器就一直不会考虑调度它们

-  【终止状态】表示线程已经执行完毕，生命周期已经结束，不会再转换为其它状态



#### 2.6.2 六种状态

这是**Java API**层面描述的

根据 Thread.State 枚举，分为六种状态

![image-20220716180530464](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207161805566.png)

- NEW 线程刚被创建，但是还没有调用 start() 方法
- RUNNABLE 当调用了 start() 方法之后，注意，Java API 层面的 RUNNABLE 状态涵盖了**操作系统层面**的【可运行状态】、【运行状态】和【阻塞状态】(由于 BIO 导致的线程阻塞，在 Java 里无法区分，仍然认为是**可运行状态**)

- BLOCKED ， WAITING ， TIMED_WAITING 都是 Java API 层面对【阻塞状态】的细分

- TERMINATED 当线程代码运行结束



# 第三章 共享模型之管程

**本章内容**

- 共享问题

- synchronized

- 线程安全分析

- Monitor 
- wait/notify 
- 线程状态转换 
- 活跃性

- Lock



## 3.1 共享存在的问题

两个线程对初始值为 0 的静态变量一个做自增，一个做自减，各做 5000 次，结果是 0 吗?

```java
static int counter = 0;
public static void main(String[] args) throws InterruptedException { 
  	Thread t1 = new Thread(() -> {
				for (int i = 0; i < 5000; i++) { 
          	counter++;
				}
		}, "t1");
		Thread t2 = new Thread(() -> {
				for (int i = 0; i < 5000; i++) {
						counter--; }
		}, "t2");
		t1.start();
		t2.start();
		t1.join();
		t2.join(); 
   	log.debug("{}",counter);
}
```

**问题分析**

以上的结果可能是正数、负数、零。为什么呢?

因为 Java 中对静态变量的自增，自减并不是原子操作，要彻底理解，必须从字节码来进行分析

例如对于 i++ 而言(i 为静态变量)，实际会产生如下的 JVM 字节码指令:

```
getstatic		i // 获取静态变量i的值
iconst_1 			// 准备常量1
iadd 					// 自增
putstatic		i // 将修改后的值存入静态变量i
```

而对应 i-- 也是类似:

```
getstatic		i // 获取静态变量i的值
iconst_1 			// 准备常量1
isub 					// 自减
putstatic		i // 将修改后的值存入静态变量i
```

而 Java 的内存模型如下，完成静态变量的自增，自减需要在主存和工作内存中进行数据交换:

![image-20220716182554299](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207161825502.png)

如果是单线程以上 8 行代码是顺序执行(不会交错)没有问题:

![image-20220716182739808](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207161827008.png)

但多线程下这 8 行代码可能交错运行:

出现负数的情况：

![image-20220716182837528](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207161828730.png)

出现正数的情况：

![image-20220716182912689](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207161829893.png)



## 3.2 临界区与竞态条件

**临界区**

- 一个程序运行多个线程本身是没有问题的 
- 问题出在多个线程访问**共享资源**
  - 多个线程读共享资源其实也没有问题 
  - 在多个线程对共享资源读写操作时发生指令交错，就会出现问题

- 一段代码块内如果存在对共享资源的多线程读写操作，称这段代码块为临界区

例如，下面代码中的临界区

```Java
static int counter = 0;
static void increment() 
// 临界区
{
counter++; 
}
static void decrement()
// 临界区
{
counter--; 
}
```



**竞态条件**

多个线程在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了竞态条件



## 3.3 解决方案

为了避免临界区的竞态条件发生，有多种手段可以达到目的。

- 阻塞式的解决方案:synchronized，Lock

- 非阻塞式的解决方案:原子变量



#### 3.3.1 synchronized

##### 概述

synchronized，即俗称的【对象锁】，它采用互斥的方式让同一 时刻至多只有一个线程能持有【对象锁】，其它线程再想获取这个【对象锁】时就会阻塞住。这样就能保证拥有锁 的线程可以安全的执行临界区内的代码，不用担心线程上下文切换



**注意**

虽然 java 中**互斥**和**同步**都可以采用 synchronized 关键字来完成，但它们还是有区别的:

- 互斥是保证临界区的竞态条件发生，同一时刻只能有一个线程执行临界区代码 

- 同步是由于线程执行的先后、顺序不同、需要一个线程等待其它线程运行到某个点

**语法**

```java
synchronized(对象) // 线程1， 线程2(blocked) 
{
		临界区
}
```

##### 解决

```java
static int counter = 0;
static final Object room = new Object();
public static void main(String[] args) throws InterruptedException { 
  	Thread t1 = new Thread(() -> {
				for (int i = 0; i < 5000; i++) { 
          	synchronized (room) {
								counter++; 
            }
				}
		}, "t1");
		Thread t2 = new Thread(() -> {
				for (int i = 0; i < 5000; i++) {
						synchronized (room) { 
              counter--;
						} 
        }
}, "t2");
		t1.start();
		t2.start();
		t1.join();
		t2.join(); 
  log.debug("{}",counter);
}
```



 **流程**

![image-20220716192149623](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207161921772.png)



##### 面向对象改进

```java
class Room {
		int value = 0;
		public void increment() { 
      	synchronized (this) {
						value++; 
        }
		}
		public void decrement() { 
      	synchronized (this) {
						value--; 
        }
		}
		public int get() { 
      	synchronized (this) {
						return value; 
        }
		} 
}
@Slf4j
public class Test1 {
		public static void main(String[] args) throws InterruptedException { 
      	Room room = new Room();
				Thread t1 = new Thread(() -> {
						for (int j = 0; j < 5000; j++) { 
              	room.increment();
						}
				}, "t1");
				Thread t2 = new Thread(() -> {
						for (int j = 0; j < 5000; j++) {
								room.decrement(); 
            }
				}, "t2"); 
      	t1.start(); 
      	t2.start();
				t1.join();
				t2.join();
				log.debug("count: {}" , room.get());
		} 
}
```

**方法上的synchronized**

```java
class Test{
		public synchronized void test() {
      
		} 
}
等价于
class Test{
		public void test() {
				synchronized(this) { 
        
        }
		} 
}
```

```java
class Test{
 		public synchronized static void test() {
      
		} 
}
等价于
class Test{
		public static void test() {
				synchronized(Test.class) { 
        
        }
		} 
}
```



#####  原理

```JAVA
static final Object lock = new Object(); 
static int counter = 0;
public static void main(String[] args) { 
  	synchronized (lock) {
				counter++; 
    }
}
```

对应的字节码为

```JAVA
public static void main(java.lang.String[]); 
		descriptor: ([Ljava/lang/String;)V 
    flags: ACC_PUBLIC, ACC_STATIC
Code:
	stack=2, locals=3, args_size=1
		0: getstatic #2 							// <- lock引用 (synchronized开始)
    3: dup
		4: astore_1										// lock引用 -> slot 1
		5: monitorenter								// 将 lock对象 MarkWord 置为 Monitor 指针
		6: getstatic #3								// <- i
		9: iconst_1										// 准备常数 1
		10: iadd											// +1
		11: putstatic #3 							// -> i
    14: aload_1										// <- lock引用
		15: monitorexit								// 将 lock对象 MarkWord 重置, 唤醒 EntryList
		16: goto 24 
    19: astore_2									// e -> slot 2
		20: aload_1										// <- lock引用
		21: monitorexit								// 将 lock对象 MarkWord 重置, 唤醒 EntryList 
		22: aload_2										// <- slot 2 (e)
		23: athrow										// throw e
		24: return
	Exception table:
		from to target type
				6 	16 	19 	any 
        19 	22	19 	any
	LineNumberTable: 
    line 8: 0
		line 9: 6
		line 10: 14 
    line 11: 24
	LocalVariableTable:
		Start Length Slot Name Signature
				0 25 0 args [Ljava/lang/String; 
  StackMapTable: number_of_entries = 2
		frame_type = 255 /* full_frame */
			offset_delta = 19
			locals = [ class "[Ljava/lang/String;", class java/lang/Object ] 
      stack = [ class java/lang/Throwable ]
		frame_type = 250 /* chop */ 
      offset_delta = 4
```

**分析**

首先获取作为锁的对象lock的引用，然后复制了一份存在一个临时变量中，解锁时用到

将lock对象的对象头中Mark Word部分换成Monitor对象的地址，来关联一个Monitor对象

根据临时变量的对象头中的Mark Word部分找到Monitor对象，将lock对象中的Mark Word重置成加锁前的，然后唤醒Monitor对象中的entrylist中的线程竞争锁。



**注意**

- 方法级别的 synchronized 不会在字节码指令中有所体现



##### 优化

让每个对象关联一个Monitor对象，Monitor对象才是真正意义上的锁，但是Monitor是由操作系统提供的，获取锁的成本很高，影响系统性能。因此，从Java6开始，对Synchronized关键字获取锁的方式进行了优化，引入了**偏向锁**和**轻量级锁**。

###### 轻量级锁

使用场景：一个对象被多个线程错开访问，即**多线程非竞争访问同一对象**，就可以使用**轻量级锁**进行优化。

语法：Synchronized修饰，即对使用者是透明的。

原理：

当一个线程访问Synchronized关键字修饰的obj对象时，会在该线程对应的栈中创建对应方法的栈帧，在栈帧中创建锁记录（Lock Record）对象，让锁记录指向obj对象，然后尝试使用CAS交换obj对象头中的Mark Word部分，将Mark Word的值保存在锁记录（Lock Record）对象中，然后对象头中的Mark Word部分保存锁记录对象的62位地址和2位的加锁状态（00）。



CAS操作：保证交换操作是原子的。

CAS操作失败原因：

- 当obj对象头中的Mark Word部分已经被替换成另一个线程中的锁记录的地址，即存在线程竞争情况，此时，会引发**锁膨胀**，即轻量级锁升级为重量级锁。
- 当同一个线程自己再次执行了Synchronized锁，则会再产生一个锁记录对象，该对象指向obj对象，并尝试使用CAS替换obj对象头中的Mark Word部分，此时发现，Mark Word已经存在一个锁记录地址和状态，且发现是自己这个线程中的锁记录对象，那么CAS操作失败，锁记录对象保存为null，锁记录数量+1。 

解锁流程：
		当退出Synchronized代码块时，如有取值为null的锁记录，表示有锁重入，这是重置锁记录，数量-1。
		当退出Synchronized代码块时，如发现锁记录的取值不为null，这时就会使用CAS将Mark Word的值恢复为对象

- 成功，则解锁成功。
- 失败，说明轻量级锁进行了锁膨胀升级为重量级锁，进入重量级锁解锁流程，即通过obj对象头中Mark Word部分找到Monitor对象，将Monitor对象中的Owner属性设为null并唤醒entryList中的阻塞线程。



###### 锁膨胀

 当前线程尝试对obj对象加轻量级锁，发现对象头中的Makl Word部分已经被其他线程的锁记录地址替换，此时加锁失败，进行锁膨胀。

过程：首先为obj对象申请Monitor对象，将Owner属性指向持有锁的线程，然后替换obj对象头中Mark Word部分为Monitor对象的地址和加锁状态（10），即让obj对象指向Monitor对象。然后**自己进行自旋**，若自旋后obj对象的锁仍然没有被释放，则**进入Monitor对象中entryList中**，进入阻塞状态。（此处有优化）



###### 自旋优化

重量级锁存在多线程竞争访问时，会进行自旋优化，不立即加入到Monitor对象中的entryList中，会自己循环几次，若循环期间，发现obj对象的锁被释放，那么就直接加锁，避免了阻塞，减少了线程的上下文切换发生。

- 从java6开始，自旋就是自适应的，即自旋次数是智能的，从java7开始就不能控制是否开启自旋功能。
- 由于自旋会占用cpu时间，因此在多核cpu下自旋才能发挥优势。



###### 偏向锁

由于轻量级锁存在在发生锁重入时仍然需要尝试使用CAS替换obj对象头中Mark Word部分，即每次仍然需要进行CAS检查的缺陷，因此引入了偏向锁进行优化。

- 默认情况下是启用偏向锁的，但偏向锁默认是有延迟的，可以通过添加VM参数
- -XX：BiasedLockingStartupDelay=0来禁用延迟。
- 如果没有开启偏向锁，可以通过VM参数-XX：-useBiasedLocking来禁用偏向锁。此时新建一个对象，对象头中的Mark Word二进制值的后三位为001，前面25位unused,31位hashcode,1位unused,4位age，1位biased_lock,2位加锁状态。其中它的hashcode，age都为0，只有在第一次hashcode时才会被赋值。

撤销偏向锁的情况：

- hashcode（）方法会禁用掉对象的偏向锁。

​		原因：从对象头中可以看出，在偏向锁状态时，Mark Word部分已经存不下31位的hashcode值。

​		为什么轻量级锁和重量级锁没有这个问题？

​		因为轻量级锁会把Mark Word存在锁记录（Lock Record）中，重量级锁会存在Monitor对象中 。

- 其他线程使用对象，此时偏向锁就会被置为不可偏向状态，然后升级为轻量级锁

  ![image-20220710145243984](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207171008203.png)  

- 调用wait/notify

  原因：只有重量级锁有，会升级成重量级锁。



###### 批量重偏向

由于偏向锁会存在撤销偏向的情况，对性能损耗较大。批量重偏向对这种情况进行了优化。

多个线程非竞争访问一个对象，这时偏向了T1线程的对象仍可能重新偏向T2，重偏向就是重置obj对象的对象头中的Mark Word中的Thread ID。

当撤销偏向的阈值（针对的是某个类的对象）超过20次时，即20<=X<40，就会发生批量重偏向。



###### 批量撤销

当撤销偏向的阈值超过40次时，即大于等于40，就会发生批量撤销。

从第40个对象开始，对象头中的MarkWord就为Normal。



###### 锁消除

Synchronized锁的obj对象当不存在多线程访问的情况，这时，JIT，即时编译器就会对代码进行优化。

锁消除优化默认是开启的，可以通过VM参数：-XX：-EliminateLocks



###### 锁粗化

对相同对象多次加锁，导致线程发生多次重入，可以使用锁粗化方式来优化，这不同于之前讲的细分锁的粒度。



##### 进阶

###### wait-notify

**API介绍**

- obj.wait() 让进入object监视器的线程到waitset中等待

- obj.notify() 在 object 上正在 waitSet 等待的线程中挑一个唤醒 

- obj.notifyAll() 让 object 上正在 waitSet 等待的线程全部唤醒

  
  

**原理**

![image-20220717205510453](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207172055618.png)

  

- Owner 线程发现条件不满足，调用 wait 方法，即可进入 WaitSet 变为 WAITING 状态
- BLOCKED 和 WAITING 的线程都处于阻塞状态，不占用 CPU 时间片
- BLOCKED 线程会在 Owner 线程释放锁时唤醒
- WAITING 线程会在 Owner 线程调用 notify 或 notifyAll 时唤醒，但唤醒后并不意味者立刻获得锁，仍需进入 EntryList 重新竞争
- 它们都是线程之间进行协作的手段，都属于 Object 对象的方法。必须获得此对象的锁，才能调用这几个方法

```java
final static Object obj = new Object();
public static void main(String[] args) {
		new Thread(() -> { 
      	synchronized (obj) {
						log.debug("执行...."); 
          	try {
								obj.wait(); // 让线程在obj上一直等待下去 
            } catch (InterruptedException e) {
								e.printStackTrace(); 
            }
  					log.debug("其它代码...."); 
        }
		}).start();
		new Thread(() -> { 
      	synchronized (obj) {
						log.debug("执行...."); 
          	try {
								obj.wait(); // 让线程在obj上一直等待下去 
            } catch (InterruptedException e) {
								e.printStackTrace(); 
            }
						log.debug("其它代码...."); 
        }
		}).start();
// 主线程两秒后执行
		sleep(2);
		log.debug("唤醒 obj 上其它线程"); 
  	synchronized (obj) {
		obj.notify(); // 唤醒obj上一个线程
// obj.notifyAll(); // 唤醒obj上所有等待线程 
    }
}
```

notify的一种结果

```
 20:00:53.096 [Thread-0] c.TestWaitNotify - 执行....
 20:00:53.099 [Thread-1] c.TestWaitNotify - 执行.... 
 20:00:55.096 [main] c.TestWaitNotify - 唤醒 obj 上其它线程 
 20:00:55.096 [Thread-0] c.TestWaitNotify - 其它代码....
```

notifyAll的结果

```
19:58:15.457 [Thread-0] c.TestWaitNotify - 执行.... 
19:58:15.460 [Thread-1] c.TestWaitNotify - 执行.... 
19:58:17.456 [main] c.TestWaitNotify - 唤醒 obj 上其它线程 
19:58:17.456 [Thread-1] c.TestWaitNotify - 其它代码.... 
19:58:17.456 [Thread-0] c.TestWaitNotify - 其它代码....
```



**wait()与wait(n)**

- wait() 方法会释放对象的锁，进入 WaitSet 等待区，从而让其他线程就机会获取对象的锁。无限制等待，直到 notify 为止

- wait(long n) 有时限的等待, 到 n 毫秒后结束等待，或是被 notify



**sleep(long n) 与wait(long n)** 

- sleep是 Thread 方法，而 wait 是 Object 的方法 

- sleep不需要强制和 synchronized 配合使用，但 wait 需要 和 synchronized 一起用

- sleep在睡眠的同时，不会释放对象锁的，但 wait 在等待的时候会释放对象锁

- 它们状态 TIMED_WAITING



**使用注意事项**

- notify 只能随机唤醒一个 WaitSet 中的线程，这时如果有其它线程也在等待，那么就可能唤醒不了正确的线程，称之为【虚假唤醒】

- 用 notifyAll 仅解决某个线程的唤醒问题，但使用 if + wait 判断仅有一次机会，一旦条件不成立，就没有重新 判断的机会了



###### 同步模式-保护性暂停

保护性暂停模式，即Guarded Suspension，一个线程等待另一个线程的结果

**要点：**

- 有一个结果需要从一个线程传递到另一个线程，让他们关联同一个 GuardedObject 

- 如果有结果不断从一个线程到另一个线程那么可以使用消息队列(见生产者/消费者) 

- JDK 中，join 的实现、Future 的实现，采用的就是此模式 

- 因为要等待另一方的结果，因此归类到同步模式



![image-20220717203350395](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207172033880.png)



实现

```java
class GuardedObject { 
  	private Object response;
		private final Object lock = new Object();
		public Object get() { 
      	synchronized (lock) { // 条件不满足则等待
						while (response == null) { 
              	try {
										lock.wait();
								} catch (InterruptedException e) {
										e.printStackTrace(); 
                }
						}
						return response;
				}
 		}
		public void complete(Object response) { 
      	synchronized (lock) {
						// 条件满足，通知等待线程 
          	this.response = response; 
          	lock.notifyAll();
				} 
    }
}
```



测试

```java
public static void main(String[] args) { 
  	GuardedObject guardedObject = new GuardedObject(); 
  	new Thread(() -> {
				try {
						// 子线程执行下载
						List<String> response = download(); 
          	log.debug("download complete..."); 
          	guardedObject.complete(response);
				} catch (IOException e) { 
          	e.printStackTrace();
				} 
    }).start();
		log.debug("waiting...");
		// 主线程阻塞等待
		Object response = guardedObject.get();
		log.debug("get response: [{}] lines", ((List<String>) response).size());
}
```

结果

```
08:42:18.568 [main] c.TestGuardedObject - waiting...
08:42:23.312 [Thread-0] c.TestGuardedObject - download complete... 
08:42:23.312 [main] c.TestGuardedObject - get response: [3] lines
```

 **超时时间保护性暂停模式**

```java
class GuardedObjectV2 {
		private Object response;
		private final Object lock = new Object();
		public Object get(long millis) { 
      	synchronized (lock) {
						// 1) 记录最初时间
						long begin = System.currentTimeMillis(); 
          	// 2) 已经经历的时间
						long timePassed = 0;
						while (response == null) {
								// 4) 假设 millis 是 1000，结果在 400 时唤醒了，那么还有 600 要等 
            		long waitTime = millis - timePassed;
								log.debug("waitTime: {}", waitTime);
								if (waitTime <= 0) {
										log.debug("break...");
										break;
                }
								try { 
                  	lock.wait(waitTime);
								} catch (InterruptedException e) { 
                  	e.printStackTrace();
								}
								// 3) 如果提前被唤醒，这时已经经历的时间假设为 400 
              	timePassed = System.currentTimeMillis() - begin; 
              	log.debug("timePassed: {}, object is null {}",timePassed, response == null); 
            }
            return response;
						
       }
    }
    
		public void complete(Object response) { 
  			synchronized (lock) {
						// 条件满足，通知等待线程 
      			this.response = response; 
      			log.debug("notify..."); 
      			lock.notifyAll();
				} 
		}
}
  
```



## 3.4 线程安全分析

成员变量和静态变量是否线程安全?

- 如果它们没有共享，则线程安全 
- 如果它们被共享了，根据它们的状态是否能够改变，又分两种情况
  - 如果只有读操作，则线程安全 
  - 如果有读写操作，则这段代码是临界区，需要考虑线程安全

局部变量是否线程安全?

-   局部变量是线程安全的
-   但局部变量引用的对象则未必
    - 如果该对象没有逃离方法的作用范围，它是线程安全的 
    - 如果该对象逃离方法的作用范围，需要考虑线程安全



### 3.4.1 局部变量

#### 基本类型

如果多个线程执行以下方法

```java
public static void test1() { 
		int i = 10;
		i++; 
}
```



![image-20220716205811234](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207162058500.png)



每个线程调用 test1()方法时局部变量 i，会在每个线程的栈帧内存中被创建多份，因此不存在共享



#### 引用类型

先看一个成员变量的例子

```java
class ThreadUnsafe {
		ArrayList<String> list = new ArrayList<>(); 
  	public void method1(int loopNumber) {
				for (int i = 0; i < loopNumber; i++) { 
          	// { 临界区, 会产生竞态条件 
          	method2();
						method3();
    				// } 临界区 
        }
		}
		private void method2() { 
      	list.add("1");
		}
		private void method3() { 
      	list.remove(0);
		} 
}
```

执行

```java
static final int THREAD_NUMBER = 2; 
static final int LOOP_NUMBER = 200; 
public static void main(String[] args) {
		ThreadUnsafe test = new ThreadUnsafe();
  	for (int i = 0; i < THREAD_NUMBER; i++) {
				new Thread(() -> { 
          	test.method1(LOOP_NUMBER);
				}, "Thread" + i).start(); 
    }
}
```

其中一种情况是，如果线程2 还未 add，线程1 remove 就会报错:

```
Exception in thread "Thread1" java.lang.IndexOutOfBoundsException: Index: 0, Size: 0 
				at java.util.ArrayList.rangeCheck(ArrayList.java:657)
				at java.util.ArrayList.remove(ArrayList.java:496)
				at cn.itcast.n6.ThreadUnsafe.method3(TestThreadSafe.java:35)
				at cn.itcast.n6.ThreadUnsafe.method1(TestThreadSafe.java:26)
				at cn.itcast.n6.TestThreadSafe.lambda$main$0(TestThreadSafe.java:14) 
				at java.lang.Thread.run(Thread.java:748)
```

分析:

- 无论哪个线程中的 method2 引用的都是同一个对象中的 list 成员变量 

- method3 与 method2 分析相同

![image-20220716201330504](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207162013683.png)



将 list 修改为局部变量，就不会出现以上问题

```java
class ThreadSafe {
		public final void method1(int loopNumber) {
				ArrayList<String> list = new ArrayList<>(); 
      	for (int i = 0; i < loopNumber; i++) {
						method2(list);
						method3(list); 
        }
		}
    private void method2(ArrayList<String> list) { 
      	list.add("1");
		}
		private void method3(ArrayList<String> list) { l
      	ist.remove(0);
		}
}
```

分析：

- list 是局部变量，每个线程调用时会创建其不同实例，没有共享
- 而 method2 的参数是从 method1 中传递过来的，与 method1 中引用同一个对象 
- method3 的参数分析与 method2 相同

![image-20220716201705240](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207162017422.png)



方法访问修饰符带来的思考，如果把 method2 和 method3 的方法修改为 public 会不会产生线程安全问题?

- 情况1:有其它线程调用 method2 和 method3 时**没有线程安全问题**
- 情况2:在 情况1 的基础上，为 ThreadSafe 类添加子类，子类覆盖 method2 或 method3 方法会**有线程安全问题**，即

```java
class ThreadSafe {
		public final void method1(int loopNumber) {
				ArrayList<String> list = new ArrayList<>(); 
      	for (int i = 0; i < loopNumber; i++) {
						method2(list);
						method3(list); }
				}
		private void method2(ArrayList<String> list) {
				list.add("1");
		}
		private void method3(ArrayList<String> list) { 
      	list.remove(0);
		} 
}
class ThreadSafeSubClass extends ThreadSafe{ 
  	@Override
		public void method3(ArrayList<String> list) { 
      	new Thread(() -> {
						list.remove(0); 
        }).start();
		} 
}
```

从这个例子可以看出 private 或 final 提供【安全】的意义所在，请体会开闭原则中的【闭】

### 3.4.2 常见线程安全类

- String
- Integer
- StringBuffer
- Random
- Vector
- Hashtable 
- java.util.concurrent 包下的类

这里说它们是线程安全的是指，多个线程调用它们同一个实例的**某个**方法时，是线程安全的。也可以理解为

```java
Hashtable table = new Hashtable();
new Thread(()->{ 
  	table.put("key", "value1");
}).start();

new Thread(()->{ 
  	table.put("key", "value2");
}).start();
```

- 它们的每个方法是原子的 

- 但注意它们多个方法的组合不是原子的，见后面分析

##### 组合调用

```java
Hashtable table = new Hashtable(); 
		// 线程1，线程2
		if( table.get("key") == null) {
				table.put("key", value); 
    }
```

![image-20220716204129382](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207162041642.png)

虽然每个操作是原子的，但是多个方法同时调用仍然会收到上下文切换的影响而产生线程安全问题



##### 不可变类

String、Integer 等都是不可变类，因为其内部的状态不可以改变，因此它们的方法都是线程安全的

String 有 replace，substring 等方法【可以】改变值——本质是重新生产了一个对象。



## 3.5 Monitor

#### 3.5.1 Java对象头

64位虚拟机Mark Word

![image-20220716214140864](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207162141147.png)

#### 3.5.2 工作原理

Monitor被译为**监视器**或**管程**

每个 Java 对象都可以关联一个 Monitor 对象，如果使用 synchronized 给对象上锁(重量级)之后，该对象头的

Mark Word 中就被设置指向 Monitor 对象的指针

Monitor结构如下

![image-20220716215001319](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207162150606.png)

- 刚开始 Monitor 中 Owner 为 null
- 当 Thread-2 执行 synchronized(obj) 就会将 Monitor 的所有者 Owner 置为 Thread-2，Monitor中只能有一 个 Owner
- 在 Thread-2 上锁的过程中，如果 Thread-3，Thread-4，Thread-5 也来执行 synchronized(obj)，就会进入 EntryList BLOCKED
- Thread-2 执行完同步代码块的内容，然后唤醒 EntryList 中等待的线程来竞争锁，竞争的时是非公平的
- 图中 WaitSet 中的 Thread-0，Thread-1 是之前获得过锁，但条件不满足进入 WAITING 状态的线程，后面讲 wait-notify 时会分析

**注意**

- synchronized 必须是进入同一个对象的 monitor 才有上述的效果

- 不加 synchronized 的对象不会关联监视器，不遵从以上规则

