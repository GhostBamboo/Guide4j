## Java并发编程笔记

----时间：2019/11/24

1. 一个对象里面如果有多个sync hronized方法，某一个时刻内，只要有一个线程去调用其中的一个synchronized方法，其他的线程都只能等待。换句话说，某一个时刻内，只能有唯一一个线程去访问这些synchronized方法。锁的是当前对象this，被锁定后，其他的线程都不能进入到当前对象的其他synchronized方法。

2. 加个普通方法后发现和同步锁无关。

3. synchronized实现同步的基础：Java中的每一个对象都可以作为锁。具体表现为以下三种形式：

   对于普通同步方法，锁是当前实例对象，锁的当前对象是this；

   对于同步代码块，锁是Synchronized括号里配置的对象；

   对于静态同步方法，锁是当前类的Class对象。

4. 当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。也就是说如果一个实例对象的非静态同步方法获取锁后，该实例对象的其它非静态同步方法必须等待获取锁的方法释放锁后才能获取锁，可是别的实例对象的非静态同步方法因为跟该实例对象的非静态同步方法用的是不同的锁所以无需等待该实例对象已获取锁的非静态同步方法释放锁就可以获取它们自己的锁。

5. 所有的静态同步方法用的也是同一把锁----类对象本身。两把锁是两个不同的对象，所以静态同步方法与非静态同步方法之间是不会有竞争关系的；

   但是一旦一个静态同步方法获取锁后，其他的静态同步方法都必须等待该方法释放锁后才能获取锁。

6. 线程交替操作：判断--->干活--->通知 (防止虚假唤醒)

   thread.await()  ---> thread run ---> thread.notify()

   防止虚假唤醒：不允许使用if判断，需要使用while

7. 

