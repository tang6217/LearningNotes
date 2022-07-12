# wait-notify

![image-20220710162852766](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207101628490.png)



当线程访问一个Sychronized修饰的对象时，会为对象关联一个Monitor对象，monitor对象的owner指向正在访问的线程，若线程发现当前条件不满足，就可以调用wait方法，进入monitor对象中的waitset中，线程进入waiting状态。然后其他线程持有obj对象的锁，执行完后调用notify方法或者notifyall方法，唤醒waitset中的waiting线程，这些线程就会进入entrylist中进入阻塞状态，然后唤醒entrylist中的线程进行竞争。



- Blocked和waiting状态都是阻塞状态，不占用CPU时间片





## sleep(n)和wait（n）的区别：

sleep是Thread中的方法，wait是object中的方法

sleep不需要强制和synchronized配合使用，wait需要

sleep在睡眠的同时不会释放锁，而wait会释放锁



共同点：线程状态都是timed waiting。

- 如果waitset中有多个线程，另一个线程调用notify时，是随机唤醒的。
- 使用notifyall唤醒所有在waitset中的线程，让他们加入到entryList中参与竞争，会存在虚假唤醒的情况，可以使用while（条件）循环，不满足条件继续进入waitset中。

套路：

```
synchronized（lock）{
		While(条件不成立){
			lock.wait()；
		}
		干活
}

synchronized（lock）{
	lock.	notifyall（）；
}

```

