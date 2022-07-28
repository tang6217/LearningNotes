比较1

wait 、notify、notifyall必须配合monitor对象一起使用，而park、unpark不需要

park&unpark是以线程为单位来【阻塞】和【唤醒】线程，相较于另外两个，就比较精确

park&unpark可以先unpark，而另外两个就不可以。



原理：

![image-20220711200710636](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost/assets/202207112007750.png)

每个线程都有一个自己的Parker对象，由三部分组成：_counter /_cond /_mutex。

- 调用Unsafe.unpark(thread_0)方法，设置_counter为1
- 唤醒_cond条件变量中的Thread_0
- thread_0恢复运行
- 设置_counter为0_





- 当前线程调用Unsafe.park()方法
- 检查_counter，如果为0，就会获得mutex互斥锁
- 线程进入cond条件变量阻塞
- 设置counter=0