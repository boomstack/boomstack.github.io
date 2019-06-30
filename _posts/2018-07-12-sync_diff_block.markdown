---

layout: post 

title: "synchronized关键字修饰静态方法与成员方法的区别" 

date: 2018-07-12 19:43:10 +0800 

categories: Android

---

开发中常用的加锁方式就是使用synchronized关键字，可以在以下三种情景使用：

 1. 修饰static方法
 2. 修饰成员方法
 3. 修饰代码块  
synchronized本质是一种独占锁，即某一时刻仅能有一个线程进入临界区，其他线程必须等待，处于block状态。下面以几个例子分别看下不同场景下的synchronized

## 修饰static方法

```java
public class SyncArea {
    public synchronized static void methodOne() {
        System.out.println("method one executed!");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    public synchronized static void methodTwo() {
        System.out.println("method two executed!");
    }
}
//调用处：
        new Thread(() -> SyncArea.methodOne()).start();
        new Thread(() -> SyncArea.methodTwo()).start();
```
如上所示，synchronized修饰两个static方法，假如有线程A先执行methodOne，然后线程B执行methodTwo，则线程B需等待线程A执行完毕后才能执行。这是因为**修饰static方法时，其实是为SyncArea.class这个对象加锁**，线程A获取锁后会sleep两秒，这期间是不会释放锁的，在这段时间内，线程B试图获取SyncArea.class这个锁，但是获取不到，自己处于block状态。两秒后线程A执行完毕释放锁，线程B才能获取到锁。
**由于锁的是SyncArea.class这个对象，所以以下代码与上边是相同的：**

```java
public class SyncArea {
    public static void methodOne() {
    	//修饰代码块，锁的是.class对象
        synchronized (SyncArea.class) {
            System.out.println("method one executed!");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public synchronized static void methodTwo() {
        System.out.println("method two executed!");
    }
}

```

注意：如果methodTwo没有用synchronized修饰，则不涉及同步问题，在上述情景下，线程B直接执行，不会试图获取锁。
## 修饰成员方法

```java
public class SyncArea {
    public synchronized void methodOne() {
        System.out.println("method one executed!");
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }

    public synchronized void methodTwo() {
        System.out.println("method two executed!");
    }
}
//调用处：
        final SyncArea sa = new SyncArea();
        new Thread(() -> sa.methodOne()).start();
        new Thread(() -> sa.methodTwo()).start();
```
假如线程A先执行methodOne，线程B再执行methodTwo，依然是线程A休眠两秒内线程B被阻塞（block）。这是因为**synchronized修饰成员方法时，其实是对实例对象加锁**，本例中即sa对象，线程A和B竞争这个对象锁。同理，上述代码与下边是一样的：synchronized修饰代码块，将this加锁。

```java
public class SyncArea {
    public void methodOne() {
        synchronized (this) {
            System.out.println("method one executed!");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    public synchronized void methodTwo() {
        System.out.println("method two executed!");
    }
}
```
## 修饰代码块
上边已经穿插介绍了synchronized修饰代码块的情景，它的本质是对某个对象加锁，更确切的说是这个对象的监视器锁。**synchronized后跟哪个对象就是多个线程需获取哪个对象的锁后才能进入临界区**。为了进一步理解，改写上述修饰成员变量的例子：

```java
public class SyncArea {
    private final Object obj = new Object();

    public void methodOne() {
        synchronized (obj) {
            System.out.println("method one executed!");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }

    public void methodTwo() {
        synchronized (obj) {
            System.out.println("method two executed!");
        }
    }
}
//调用处：
        final SyncArea sa = new SyncArea();
        new Thread(() -> sa.methodOne()).start();
        new Thread(() -> sa.methodTwo()).start();

```
新建一个对象，对此对象加锁，执行结果相同，依然是线程A休眠期间线程B被阻塞。与修饰成员变量方法不同的是，这次多个线程获取的锁是obj对象，不再是this，其他道理是一样的。
总结：

 **哪个线程能进入临界区，需要看synchronized究竟是对哪个对象加锁，修饰static方法是对.class对象加锁，修饰成员方法是对当前类的某个对象加锁，修饰代码块是对特定对象加锁。**
