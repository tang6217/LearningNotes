#  Synchronized

让每个对象关联一个Monitor对象，Monitor对象才是真正意义上的锁，但是Monitor是由操作系统提供的，获取锁的成本很高，影响系统性能。因此，从Java6开始，对Synchronized关键字获取锁的方式进行了优化，引入了**偏向锁**和**轻量级锁**。



![image-20220710124735833](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207101247219.png)



## 轻量级锁

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



## 锁膨胀

 当前线程尝试对obj对象加轻量级锁，发现对象头中的Makl Word部分已经被其他线程的锁记录地址替换，此时加锁失败，进行锁膨胀。

过程：首先为obj对象申请Monitor对象，将Owner属性指向持有锁的线程，然后替换obj对象头中Mark Word部分为Monitor对象的地址和加锁状态（10），即让obj对象指向Monitor对象。然后**自己进行自旋**，若自旋后obj对象的锁仍然没有被释放，则**进入Monitor对象中entryList中**，进入阻塞状态。（此处有优化）



## 自旋优化

重量级锁存在多线程竞争访问时，会进行自旋优化，不立即加入到Monitor对象中的entryList中，会自己循环几次，若循环期间，发现obj对象的锁被释放，那么就直接加锁，避免了阻塞，减少了线程的上下文切换发生。

- 从java6开始，自旋就是自适应的，即自旋次数是智能的，从java7开始就不能控制是否开启自旋功能。
- 由于自旋会占用cpu时间，因此在多核cpu下自旋才能发挥优势。



## 偏向锁

由于轻量级锁存在在发生锁重入时仍然需要尝试使用CAS替换obj对象头中Mark Word部分，即每次仍然需要进行CAS检查的缺陷，因此引入了偏向锁进行优化。

- 默认情况下是启用偏向锁的，但偏向锁默认是有延迟的，可以通过添加VM参数-XX：BiasedLockingStartupDelay=0来禁用延迟。
- 如果没有开启偏向锁，可以通过VM参数-XX：-useBiasedLocking来禁用偏向锁。此时新建一个对象，对象头中的Mark Word二进制值的后三位为001，前面25位unused,31位hashcode,1位unused,4位age，1位biased_lock,2位加锁状态。其中它的hashcode，age都为0，只有在第一次hashcode时才会被赋值。

撤销偏向锁的情况：

- hashcode（）方法会禁用掉对象的偏向锁。

​		原因：从对象头中可以看出，在偏向锁状态时，Mark Word部分已经存				不下31位的hashcode值。

​		为什么轻量级锁和重量级锁没有这个问题？

​		因为轻量级锁会把Mark Word存在锁记录（Lock Record）中，重量级				锁会存在Monitor对象中 。

- 其他线程使用对象，此时偏向锁就会被置为不可偏向状态，然后升级为轻量级锁

  ![image-20220710145243984](https://cdn.jsdelivr.net/gh/tang6217/MyImageHost@master//assets/202207101452027.png)  

- 调用wait/notify

  原因：只有重量级锁有，会升级成重量级锁。



## 批量重偏向

由于偏向锁会存在撤销偏向的情况，对性能损耗较大。批量重偏向对这种情况进行了优化。

多个线程非竞争访问一个对象，这时偏向了T1线程的对象仍可能重新偏向T2，重偏向就是重置obj对象的对象头中的Mark Word中的Thread ID。

当撤销偏向的阈值（针对的是某个类的对象）超过20次时，即20<=X<40，就会发生批量重偏向。



## 批量撤销

当撤销偏向的阈值超过40次时，即大于等于40，就会发生批量撤销。

从第40个对象开始，对象头中的MarkWord就为Normal。



## 锁消除

##

Synchronized锁的obj对象当不存在多线程访问的情况，这时，JIT，即时编译器就会对代码进行优化。

锁消除优化默认是开启的，可以通过VM参数：-XX：-EliminateLocks
