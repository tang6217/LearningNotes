#    ReentrantLock

相较于sychronized它具备如下特点：

- 可中断

- 可以设置超时时间

- 可以设置为公平锁

- 支持多个条件变量

  与sychronized一样，都支持可重入



基本语法

```java
// 获取锁
reentrantLock.lock();
try {
	// 临界区
} finally {
	// 释放锁
	reentrantLock.unlock();
}
```





## 可重入

同一个线程可以对以拥有的锁再次获取。（原理：线程对应栈中会产生锁记录（Lock Record））

```java
@Slf4j(topic = "c.Test22")
public class Test22 {
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        
        lock.lock();
        try {
			// 临界区
            log.debug("获得到锁");
            m1();
		} finally {
			// 释放锁
			lock.unlock();
		}
        
        
        public static void m1() {
        	lock.lock();
        	try {
				// 临界区
            	log.debug("获得到锁");
            	m2();
			} finally {
				// 释放锁
				lock.unlock();
			}
        }
        
        
        public static void m2() {
        	lock.lock();
        	try {
				// 临界区
            	log.debug("获得到锁");
            	m2();
			} finally {
				// 释放锁
				lock.unlock();
			}
        }
    }

}
```



## 可中断

B线程可以打断因竞争失败而陷入阻塞队列的线程A

```java
@Slf4j(topic = "c.Test")
public class Test {
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            try {
                // 如果没有竞争那么此方法就会获取到lock对象锁
                // 如果有竞争就进入阻塞队列，可以被其它线程用interrupt() 方法打断
                lock.lockInterruptibly;
            } catch (InterruptedException e) {
                e.printStackTrace();
                log.debug("获取不到锁");
                return;
            }
            try {
                log.debug("获得到锁");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        log.debug("获得到锁");
        t1.start();
        
        sleep(1);
        log.debug("打断t1");
        /**
        *如果被打断线程正在 sleep，wait，join 会导致被打断
		*的线程抛出 InterruptedException，并清除 打断标
		*记 ；如果打断的正在运行的线程，则会设置 打断标
		*记 ；park 的线程被打断，也会设置 打断标记
        */
        
        t1.interrupt();
    }

}
```



## 锁超时

```java
@Slf4j(topic = "c.Test")
public class Test {
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        Thread t1 = new Thread(() -> {
            log.debug("尝试获得锁");
            try {
                if (! lock.tryLock(2, TimeUnit.SECONDS)) {
                    log.debug("获取不到锁");
                    return;
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
                log.debug("获取不到锁");
                return;
            }
            try {
                log.debug("获得到锁");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        log.debug("获得到锁");
        t1.start();
        sleep(1);
        log.debug("释放了锁");
        lock.unlock();
    }

}
```

## 公平锁

ReentranLock默认是不公平的，可以在创建时设置为true

```java
    /**
     * Creates an instance of {@code ReentrantLock} with the
     * given fairness policy.
     *
     * @param fair {@code true} if this lock should use a fair ordering policy
     */
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

公平锁一般没有必要，会降低并发度。



## 条件变量

synchronized中也有条件变量，当条件不满足时进入waitset中等待

ReentrantLock支持多个条件变量

使用流程：

- await前需要获得锁
- await执行后，会释放锁，进入conditionObject等待
- await的线程被唤醒（或被打断、或超时），则重新竞争lock锁
- 竞争lock锁成功后，从await后继续执行



