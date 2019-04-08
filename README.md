# JavaMultiThreadProgramming_Study

## 作者：冰红茶  
## 参考书籍：《Java核心技术》卷一和卷二 原书第10版 《Java多线程编程核心技术》
    
------    
    
记得一年前由于项目需要，急匆匆的拿一本《Java从入门到精通》边学边写StarCCM+的插件。该书实战性很强，就是内容比较浅薄。所以今天又啃起了编程思想，断断续续花了两周时间看了大概看了一半，剩余的内容后面将由JVM和多线程和Hadoop三本书补充完整。大数据的基础是Java和Linux，以后尽管进不了大数据的领域，也希望能借助此基础进入后端领域^_ ^

## 目录
## [一、线程](#1)
### [1.1 什么是线程](#1.1)
### [1.2 线程状态与线程属性](#1.2)
## [二、同步](#2)
### [2.1 synchronized](#2.1)
### [2.2 线程安全与非线程安全](#2.2)
### [2.3 同步](#2.3)
### [2.4 volatile关键字](#2.4)
## [三、线程间通讯](#3)
### [3.1 等待/通知机制](#3.1)
### [3.2 通过管道进行线程间通讯](#3.2)
### [3.3 方法join 类ThreadLocal和类inheritableThreadLocal](#3.3)
## [四、Lock的使用](#4)
### [4.1 等待/通知机制](#4.1)
### [4.2 通过管道进行线程间通讯](#4.2)
        
------      
        
<h2 id='1'>一、线程</h2>
<h3 id='1.1'>1.1 什么是线程</h3>  
        
#### 1) 介绍
> - 通常，每一个任务称为一个线程，可以同时运行一个以上线程的程序称为多线程程序
> - 线程和进程的区别：每个进程拥有自己一整套变量，而线程则共享数据。共享变量使得线程之间的通讯比进程之间的通讯更加有效和容易，与进程相比，线程更加轻量级，创建，撤销一个线程比启动新进程的开销要小得多。
> - 可以理解为线程是进程中独立运行的子任务。
#### 2) 如何开启一个线程
> - 方法一：利用Runnable接口创建Thread类
> - 方法二：继承Thread类然后重写run()方法
> - 方法二：实现Runnable接口然后重写run()方法
> - 接口比继承要好的一个原因是，Java中只允许单根继承。何为单根继承，就是每次只能继承一个超类，而接口可以实现多个实现。
> - 不要直接调用Thread类或者Runnable对象的run方法，因为直接调用run方法只会执行同一个线程中的任务，而不会开启新的线程。开启新的线程应该调用Thread.start()方法
> - 值得注意的是，执行Thread.start()方法顺序不代表线程启动的顺序
                
                /**
                 * 
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker {
                    
                    Runnable r;
                    Thread t;
                    /**
                     * @param r
                     * @param t
                     */
                    public Worker(Runnable r) {
                        this.r = r;
                        this.t = new Thread(r);
                    }

                    public void start() {
                        t.start();
                    }
                    
                    public void run() {
                        t.run();
                    }
                }

#### 3) 中断线程
> - 两种结束的条件
>> - 当run方法执行到最后一条语句后，并经由执行return语句返回时
>> - 抛出异常 一般跟sleep()一同使用，在try-catch方法体中遇到异常立马进行catch(),try()中sleep()后面的语句不再运行。而且catch()里面的代码将放在最后进行
>> - 使用Stop方法或者suspend暴力终止，但是不建议使用这个已经被废弃的方法。调用这个方法会抛出java,lang.ThreadDeath异常，但是通常情况下，此异常不需要显示地捕捉。另外，如果强制让线程停止则有可能使一些清理性的工作得不到完成。另外一种情况是对锁定的对象进行了“解锁”，导致数据得不到同步的处理，出现数据不一致的问题。
>> - 使用interrupt()方法中断，但是这个方法不能完全杀死，仅仅是在当前线程中打了一个停止的标记，然后可以通过判断标志位 + return跳出线程
> - 在早期的Java版本中可以使用stop()方法进行线程终结，但是这个方法已经被弃用了，因为“没有任何语言方面的需求要求一个被中断的线程应该终止，中断一个程序不过是引起它的注意，被中断的线程可以决定如何相应中断”
> - 每个线程都有一个Boolean状态叫“中断状态”
>> - 使用静态的Thread.currentThread()方法可以获取当前的线程
>> - 然后使用isInterrupt()方法可以检查这个状态的标志位以判断线程是否被中断。但是如果线程被阻塞或者被中断的时候调用了sleep()或者wait()方法，就会由于无法检测而产生InterruptException异常
                
                // 这两种方法都可以用来检测线程是都被中断，但是前者是对线程的标志位是有影响的，使用后就会清除该线程的中断状态，中断状态会恢复为false；而后者则是无损的

                Thread.currentThread().interrupted();     //Tests whether the current thread has been interrupted. The interrupted status of the thread is cleared by this method. Inother words, if this method were to be called twice in succession, thesecond call would return false (unless the current thread wereinterrupted again, after the first call had cleared its interruptedstatus and before the second call had examined it).    

                Thread.currentThread().isInterrupted(); //Tests whether this thread has been interrupted. The interruptedstatus of the thread is unaffected by this method. 
>> - 使用interrupt()方法可以进行中断置位
                
                Thread.currentThread().interrupt();

> - 一个例子
                
                /**
                 * 
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker {
                    
                    Runnable r;
                    Thread t;
                    /**
                     * @param r
                     * @param t
                     */
                    public Worker(Runnable r) {
                        this.r = r;
                        this.t = new Thread(r);
                    }

                    public void start() {
                        t.start();
                    }
                    
                    public void run() {
                        t.run();
                    }
                }

                Runnable r = () -> {
                    System.out.println("start");
                    int i = 0;
                    try {
                        while(!Thread.currentThread().isInterrupted()) {
                            System.out.println("打开一个新的线程"+ i);
                            i++;
                            if(i >= 10) {
                                Thread.currentThread().interrupt();
                                Thread.sleep(1000);
                            }  
                        }
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }finally {
                        System.out.println("close!");
                    }
                };

                Worker w = new Worker(r);
                w.start();

                start
                打开一个新的线程0
                打开一个新的线程1
                打开一个新的线程2
                打开一个新的线程3
                打开一个新的线程4
                打开一个新的线程5
                打开一个新的线程6
                打开一个新的线程7
                打开一个新的线程8
                打开一个新的线程9
                java.lang.InterruptedException: sleep interrupted
                    at java.base/java.lang.Thread.sleep(Native Method)
                    at practice.HelloWorld.lambda$0(HelloWorld.java:82)
                    at java.base/java.lang.Thread.run(Thread.java:844)
                close!
#### 3) 中断语句顺序
> - 同步代码首先执行，异步最后执行
> - 同步线程加上sleep()方法后变成异步，异步之间看顺序
> - 遇到异常一般是异常内部的语句最后执行
#### 4) 暂停线程
> - suspend() 暂停线程
> - resume() 重启线程
> - 值得注意的是问题：
>> - 无法访问公共的同步对象：如果使用不当，极易造成公共的同步对象的占用，使得其他线程无法访问呢公共的同步对象，异步的顺序也会被跳过
>> - println()的同步问题：println()方法是同步的，加入你在被暂停的线程中出现println()方法，那么其他地方的println()方法也会被暂停
>> - 数据有时候无法同步，比如需要给两个或以上的实例域进行赋值时，中间突然被暂停，那么后续的值无法赋值了
#### 5) 其他API
> - Thread.currentThread() 使用静态的方法获取当前线程
> - isAlive()判断线程是否处于活动状态
> - join()等待线程终止后做某事
> - sleep(Long long) 线程休眠一段时间后再重启
> - getName() 获取线程的名字
> - getId() 取得线程的位移辨识
> - setDaemon(Boolean boolean) 设置守护线程
> - yield()方法 作用是放弃当前的资源，将他让给其他任务去占用CPU执行时间，但是放弃的时间不确定，当其他资源富余的时候再继续，一般这样一搞该线程的运行时间会明显增加

        
<h3 id='1.2'>1.2 线程状态与属性</h3>  
        
#### 1) 六种状态
> - New(新创建) 使用new进行创建的时候
> - Runnable(可运行) 调用start()方法后
> - Blocked(被阻塞)
>> - 等待获得内部的对象锁
>> - 等待条件满足
>> - 等待超时参数
> - Waiting(等待) 
> - Timed waiting(计时等待)
> - Termianted(被终止)  
>>>>>> ![图1-1 线程六种状态.jpg](https://github.com/hblvsjtu/JavaMultiThreadProgramming_Study/blob/master/picture/%E5%9B%BE1-1%20%E7%BA%BF%E7%A8%8B%E5%85%AD%E7%A7%8D%E7%8A%B6%E6%80%81.jpg?raw=true)
            
#### 2) 线程属性
> - 线程的优先级 可以使用setPriority方法提高或者降低任何一个线程的优先级，优先等级从1~10从低到高，普通的是5。这个设置优先级要慎重，因为如果有几个高优先级的线程没有进入活动状态，那么低优先级的线程就不会运行而被“饿死”。但是不是说一定要等等级高的执行完才执行优先级低的线程，只是说优先级高的线程所获得的资源比较多
> - 线程的优先级以及继承，也就说用线程A去启动线程B，那么这两个线程的优先级相同
> - 优先级具有随机性，有时候低优先级的线程可能先执行完。
> - 守护线程 其实就是后台线程 可以使用 Thread.currentThread().setDaemon(true);进行设置。只要JVM中有非守护线程，那么守护线程就不会被回收，就像是“保姆”一样，最典型的应用就是GC

        
------      
        
<h2 id='2'>二、同步</h2>
<h3 id='2.1'>2.1 synchronized</h3>
            
#### 1) 对象锁
> - 是对象锁而非代码锁
> - 是保证某个对象实例中的某个方法的操作的同步，类似println()的同步特性，而同一个对象的异步方法则可以继续异步
> - “只有共享资源的读写访问才需要同步化，如果不是共享资源，那么根本没有同步的必要”
#### 2) 可重入锁
> - 自己可以再次获得自己的内部锁
> - 意思就是说一条线程获得了某个对象的锁，此时这个对象锁（对象实例内部某个synchronized方法）还没有释放，还可以再次获取这个独享的锁（对象实例内部另一个synchronized方法）
#### 3) 异常抛出
> - 异常抛出时，对象锁会自动释放
#### 4) 继承性
> - 同步不可继承，即便是覆写也需要再添加synchronized关键字
#### 5) 同步代码块
> - 可以缓解由于同步造成效率低下的问题
> - 解决的办法是在一个方法中对一部分代码进行同步的操作，而另一部分的代码还是原来的异步操作
> - “对象监视器”：“当一个线程访问object的一个synchronized同步代码块时，其他线程对同一个object中其他所有的synchronized(this)同步代码块的访问将被堵塞，也就是说同一个object中其他所有的synchronized(this)同步代码块也是同步的”
                
                synchronized(this) {
                    // todo something
                }
#### 6) 锁非this对象
> - 除了可以锁this，还可以锁其他任意的对象作为“对象监视器”
> - 优点：synchronized(非this)代码块中的程序和同步方法是异步的，不与其他锁的this同步方法争抢this锁，则可以大大提高效率
> - 只要对象不变，即使对象的属性被改变，运行的结果还是同步的
>>>>>> ![图1-2 同步代码块的三个结论.jpg](https://github.com/hblvsjtu/JavaMultiThreadProgramming_Study/blob/master/picture/%E5%9B%BE1-2%20%E5%90%8C%E6%AD%A5%E4%BB%A3%E7%A0%81%E5%9D%97%E7%9A%84%E4%B8%89%E4%B8%AA%E7%BB%93%E8%AE%BA.jpg?raw=true)

#### 6) 添加static
> - 从表现来看，加跟不加没有什么本质的区别
> - 但是还是有区别的，添加static是对Class类上锁，而不加的话是对对象实例上锁。
        
<h3 id='2.2'>2.2 线程安全与非线程安全</h3>  
        
#### 1) 影响线程安全的三个因素
> - 原子性
> - 可见性
> - 随机性
#### 2) 不共享数据的情况
> - 创建多个实例，不同实例之间肯定是不会共享数据的
                
                /**
                 * 相同的线程类
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker {
                    
                    Runnable r;
                    Thread t;
                    /**
                     * @param r
                     * @param t
                     */
                    public Worker(Runnable r) {
                        this.r = r;
                        this.t = new Thread(r);
                    }

                    public void start() {
                        t.start();
                    }
                    
                    public void run() {
                        t.run();
                    }
                }

                Runnable r = () -> {
                    System.out.println("start");
                    int i = 0;
                    try {
                        
                        while(!Thread.currentThread().isInterrupted()) {
                            System.out.println("打开一个新的线程"+ i);
                            i++;
                            if(i >= 10) {
                                Thread.currentThread().interrupt();
                                Thread.sleep(1000);
                            }  
                        }
                    } catch (InterruptedException e) {
                        // TODO Auto-generated catch block
                        e.printStackTrace();
                    }finally {
                        System.out.println("close!");
                    }
                };
                Worker w1 = new Worker(r);
                Worker w2 = new Worker(r);
                Worker w3 = new Worker(r);
                Worker w4 = new Worker(r);
                Worker w5 = new Worker(r);
                w1.start();
                w2.start();
                w3.start();
                w4.start();
                w5.start();

                start
                start
                start
                打开一个新的线程0
                打开一个新的线程0
                start
                打开一个新的线程1
                打开一个新的线程1
                start
                打开一个新的线程0
                打开一个新的线程0
                打开一个新的线程2
                打开一个新的线程2
                打开一个新的线程0
                打开一个新的线程3
                打开一个新的线程3
                打开一个新的线程1
                打开一个新的线程1
                打开一个新的线程2
                打开一个新的线程4
                打开一个新的线程4
                打开一个新的线程1
                打开一个新的线程5
                打开一个新的线程5
                打开一个新的线程3
                打开一个新的线程2
                打开一个新的线程4
                打开一个新的线程6
                打开一个新的线程6
                打开一个新的线程2
                打开一个新的线程7
                打开一个新的线程7
                打开一个新的线程5
                打开一个新的线程6
                打开一个新的线程3
                打开一个新的线程7
                打开一个新的线程8
                打开一个新的线程8
                打开一个新的线程3
                打开一个新的线程9
                打开一个新的线程9
                打开一个新的线程8java.lang.InterruptedException: sleep interrupted

                打开一个新的线程4
                打开一个新的线程9
                打开一个新的线程4
                打开一个新的线程5
                打开一个新的线程5
                打开一个新的线程6
                打开一个新的线程6
                打开一个新的线程7
                打开一个新的线程7
                打开一个新的线程8
                打开一个新的线程8
                打开一个新的线程9
                打开一个新的线程9
                    at java.base/java.lang.Thread.sleep(Native Method)
                    at practice.HelloWorld.lambda$0(HelloWorld.java:83)
                    at java.base/java.lang.Thread.run(Thread.java:844)
                close!
                java.lang.InterruptedException: sleep interrupted
                    at java.base/java.lang.Thread.sleep(Native Method)
                    at practice.HelloWorld.lambda$0(HelloWorld.java:83)

        
#### 3) 共享数据但不安全的情况
> - 共享数据但不安全的情况，称为“非线程安全”，主要指多个线程对同一个对象中的同一个实例变量（或者叫“实例域”）进行操作的时候会出现值被更改，值不同步的情况，进而影响程序的执行流程。即所谓“脏读”
> - 而如果更改的是方法中的局部变量则不会存在线程不安全的问题，因为变量都不可能共享
                
                // 实现Runnable接口的版本
                /**
                 * 
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker implements Runnable {
                    
                    private int i;

                    /**
                     * @param i
                     */
                    public Worker(int i) {
                        super();
                        this.i = i;
                    }

                    /**
                     * @return the i
                     */
                    public int getI() {
                        return i;
                    }

                    /**
                     * @param i the i to set
                     */
                    public void setI(int i) {
                        this.i = i;
                    }

                    @Override
                    public void run() {
                        // TODO Auto-generated method stub
                //      super.run();
                        this.i++;
                        System.out.println("由 " + Thread.currentThread().getName() + "计算，i的值为 " + i);
                    }
                    
                }

                // 继承Thread的版本
                /**
                 * 
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker extends Thread {
                    
                    private int i;

                    /**
                     * @param i
                     */
                    public Worker(int i) {
                        super();
                        this.i = i;
                    }

                    /**
                     * @return the i
                     */
                    public int getI() {
                        return i;
                    }

                    /**
                     * @param i the i to set
                     */
                    public void setI(int i) {
                        this.i = i;
                    }

                    @Override
                    public void run() {
                        // TODO Auto-generated method stub
                        super.run();
                        this.i++;
                        System.out.println("由 " + Thread.currentThread().getName() + "计算，i的值为 " + i);
                    }
                    
                }

                // 执行类
                Worker w = new Worker(0);
                Thread t1 = new Thread(w, "t1");
                Thread t2 = new Thread(w, "t2");
                Thread t3 = new Thread(w, "t3");
                Thread t4 = new Thread(w, "t4");
                Thread t5 = new Thread(w, "t5");
                t1.start();
                t2.start();
                t3.start();
                t4.start();
                t5.start();

                // 控制台
                由 t1计算，i的值为 1
                由 t3计算，i的值为 2
                由 t4计算，i的值为 4
                由 t5计算，i的值为 4
                由 t2计算，i的值为 5

        
#### 4) 共享数据且线程安全的情况
> - 共享数据且线程安全，加关键字synchronized 使得多个线程在执行run()方法的时候，以排队的方式进行。
> - 或者在此版本的代码中，我发现只要将数据变化的那一部分放在打印语句中，就不会出现“非线程安全”。这是因为Println()这个方法本书就是同步的。
                
                /**
                 * 
                 */
                package practice;

                /**
                 * @author LvHongbin
                 *
                 */
                public class Worker extends Thread {
                    
                    private int i;

                    /**
                     * @param i
                     */
                    public Worker(int i) {
                        super();
                        this.i = i;
                    }

                    /**
                     * @return the i
                     */
                    public int getI() {
                        return i;
                    }

                    /**
                     * @param i the i to set
                     */
                    public void setI(int i) {
                        this.i = i;
                    }

                    @Override
                    synchronized public void run() {
                        // TODO Auto-generated method stub
                        super.run();
                        this.i++;
                        System.out.println("由 " + Thread.currentThread().getName() + "计算，i的值为 " + i);
                    }
                    
                }

                // 执行类
                Worker w = new Worker(0);
                Thread t1 = new Thread(w, "t1");
                Thread t2 = new Thread(w, "t2");
                Thread t3 = new Thread(w, "t3");
                Thread t4 = new Thread(w, "t4");
                Thread t5 = new Thread(w, "t5");
                t1.start();
                t2.start();
                t3.start();
                t4.start();
                t5.start();

                // 控制台
                由 t2计算，i的值为 1
                由 t5计算，i的值为 2
                由 t1计算，i的值为 3
                由 t3计算，i的值为 4
                由 t4计算，i的值为 5
#### 5) 其他非线程安全的情况
> - i++ 不是原子操作
>> - 从内存中取出i的值
>> - 计算i的值
>> - 将i的值写道内存中
> - 使用AtomicInteger类可以避免i++的非原子性，其实就是原子性的加一操作。但是使用原子性变量也不能完全保证原子性，因为还有随机性，随机性的保证还是需要同步去解决
                
                private AtomicInteger i1 = new AtomicInteger(0);
                i1.incrementAndGet();

        
<h3 id='2.3'>2.3 多线程的死锁</h3>  
                             
#### 1) 一个死锁的case
> - 程序设计的时候应该避免双方互相持有对象的锁的情况
>>>>>> ![图1-3 死锁的case.jpg](https://github.com/hblvsjtu/JavaMultiThreadProgramming_Study/blob/master/picture/%E5%9B%BE1-3%20%E6%AD%BB%E9%94%81%E7%9A%84case.jpg?raw=true)
                
        
<h3 id='2.4'>2.4 volatile关键字</h3>  
        
#### 1) 主要作用
> - 使变量在多个线程中可见，强制从公共堆栈中取得变量的值，而不是从线程私有数据栈中取得变量的值
#### 2) 跟synchronized的比较
> - volatile关键字解决的是变量在多个线程中的可见性，但不能保证原子性；而synchronized关键字解决的是多个线程之间的访问资源的同步性，也可以间接保证可见性
> - volatile关键字只能修饰变量，而synchronized关键字可以修饰方法
> - volatile关键字不会发生堵塞，而synchronized关键字会出现堵塞
#### 3) volatile的非原子特性
> - 变量在内存中的工作过程图
![1-4 变量在内存中的工作过程.jpg](https://github.com/hblvsjtu/JavaMultiThreadProgramming_Study/blob/master/picture/%E5%9B%BE1-4%20%E5%8F%98%E9%87%8F%E5%9C%A8%E5%86%85%E5%AD%98%E4%B8%AD%E7%9A%84%E5%B7%A5%E4%BD%9C%E8%BF%87%E7%A8%8B.jpg?raw=true)
                

        
------      
        
<h2 id='3'>三、线程间通讯</h2>
<h3 id='3.1'>3.1 等待/通知机制</h3>
            
#### 1) 不使用等待/通知机制实现线程间通讯
> - 使用轮询机制 + 休眠中断
> - 弊端：轮询机制非常的消耗资源
                
                // ThreadA类
                package practice;

                import java.util.ArrayList;
                import java.util.List;

                /**
                 * @author LvHongbin
                 *
                 */
                public class ThreadA implements Runnable {

                    private List<Integer> myList;

                    /**
                     * @param myList
                     */
                    public ThreadA() {
                        super();
                        this.myList = new ArrayList<Integer>();
                    }
                    

                    /**
                     * @return the list
                     */
                    public List<Integer> getMyList() {
                        return myList;
                    }


                    /**
                     * @param list the list to set
                     */
                    public void setList(List<Integer> myList) {
                        this.myList = myList;
                    }


                    @Override
                    public void run() {
                        for(int i=0; i<10; i++) {
                            this.myList.add(i);
                            System.out.println(Thread.currentThread().getName() + " 添加了元素：" + i);
                        }
                    }
                }

                // ThreadB类
                /**
                 * 
                 */
                package practice;

                import java.util.List;

                /**
                 * @author LvHongbin
                 *
                 */
                public class ThreadB implements Runnable{

                    /**
                     * 
                     */
                    private List<Integer> myList;

                    /**
                     * @param myList
                     */
                    public ThreadB(List<Integer> myList) {
                        super();
                        this.myList = myList;
                    }

                    @Override
                    public void run() {
                        int length = 0;
                        try {
                            while(true) {
                                if(this.myList.size() > 5) {
                                    length = this.myList.size();
                                    Thread.currentThread().interrupt();
                                    throw new InterruptedException();
                                }
                            }
                        }catch(InterruptedException e) {
                            e.printStackTrace();
                            System.out.println(Thread.currentThread().getName() + " 已经被停止，停止时元素数量为： " + length);
                        }
                    }
                }

                // 执行类
                ThreadA a = new ThreadA();
                ThreadB b = new ThreadB(a.getMyList());
                Thread ta = new Thread(a, "ThreadA");
                Thread tb = new Thread(b, "ThreadB");
                ta.start();
                tb.start();

                // 控制台
                ThreadA 添加了元素：0
                ThreadA 添加了元素：1
                ThreadA 添加了元素：2
                ThreadA 添加了元素：3
                ThreadA 添加了元素：4
                ThreadA 添加了元素：5
                ThreadA 添加了元素：6
                ThreadA 添加了元素：7
                ThreadA 添加了元素：8
                ThreadA 添加了元素：9
                java.lang.InterruptedException
                    at practice.ThreadB.run(ThreadB.java:35)
                    at java.base/java.lang.Thread.run(Thread.java:844)
                ThreadB 已经被停止，停止时元素数量为： 6

#### 2) 使用等待/通知机制实现线程间通讯
> - 两者所在线程必须获得对象级别锁，否则会抛出IllegalMonitorStateException，它是RuntimeException的子类，所以不需要try-catch语句进行捕捉
> - wait使线程停止运行，释放对象锁，在从wait方法返回前，线程需要与其他线程竞争重新获得锁
> - 而notify使停止的线程继续运行。不是说运行notify就马上生效，而是要等notify所在的线程运行完后，当前线程才会释放锁
> - 关键字synchronized把每个Object对象看作同步对象，而Java为每个Object都实现了wait()和notify()方法
> - notify()每次只通知一个线程，如果有多个线程的话可以使用notifyAll()，优先级最高的那个最先被执行，但也可能会随机执行，因为这主要取决于JVM虚拟机的实现
> - 注意sleep()不释放锁，他是同步的效果
> - 在等待的时候遇到异常会提前释放锁，wait后面的代码不再执行
> - wait(long time)等待一定的时间，如果超过规定的时间则自动唤醒
> - 如果通知过早了，那么wait()就永远唤不醒了
                
                package practice;

                import java.util.ArrayList;
                import java.util.List;

                /**
                 * @author LvHongbin
                 * ThreadA 类
                 */
                public class ThreadA implements Runnable {

                    private List<Integer> myList;

                    /**
                     * @param myList
                     */
                    public ThreadA() {
                        super();
                        this.myList = new ArrayList<Integer>();
                    }
                    

                    /**
                     * @return the list
                     */
                    public List<Integer> getMyList() {
                        return myList;
                    }


                    /**
                     * @param list the list to set
                     */
                    public void setMyList(List<Integer> myList) {
                        this.myList = myList;
                    }

                    @Override
                    public void run() {
                        System.out.println(Thread.currentThread().getName() + " 开启");
                        synchronized(myList) {
                            System.out.println(Thread.currentThread().getName() + " 进入对象锁的第一行");
                            for(int i=0; i<10; i++) {
                                this.myList.add(i);
                                System.out.println(Thread.currentThread().getName() + " 添加了元素：" + i);
                                if (this.myList.size() == 5) {
                                    myList.notify();
                                    System.out.println("已经发出通知，相应的线程被停止，停止时元素数量为： " + i);
                                }
                            }
                            System.out.println(Thread.currentThread().getName() + " 退出对象锁");
                        }
                        System.out.println(Thread.currentThread().getName() + " 退出");
                    }
                }


                
                package practice;

                import java.util.List;

                /**
                 * @author LvHongbin
                 * ThreadB 类
                 */
                public class ThreadB implements Runnable{

                    /**
                     * 
                     */
                    private List<Integer> myList;

                    /**
                     * @param lock
                     */
                    public ThreadB() {
                        super();
                    }


                    /**
                     * @return the myList
                     */
                    public List<Integer> getMyList() {
                        return myList;
                    }


                    /**
                     * @param myList the myList to set
                     */
                    public void setMyList(List<Integer> myList) {
                        this.myList = myList;
                    }


                    @Override
                    public void run() {
                        System.out.println(Thread.currentThread().getName() + " 开启 ");
                        synchronized (myList) {
                            System.out.println(Thread.currentThread().getName() + " 进入对象锁的第一行");
                            try {
                                myList.wait();
                                Thread.currentThread().interrupt();
                                throw new InterruptedException();
                            }catch(InterruptedException e) {
                                e.printStackTrace();
                            }
                            System.out.println(Thread.currentThread().getName() + " 退出对象锁");
                        }
                        System.out.println(Thread.currentThread().getName() + " 退出");
                    }
                }


                // 执行类
                ThreadA a = new ThreadA();
                ThreadB b = new ThreadB();
                b.setMyList(a.getMyList());
                Thread ta = new Thread(a, "ThreadA");
                Thread tb = new Thread(b, "ThreadB");
                tb.start();
                ta.start();

                // 控制台
                ThreadB 进入对象锁的第一行
                ThreadA 进入对象锁的第一行
                ThreadA 添加了元素：0
                ThreadA 添加了元素：1
                ThreadA 添加了元素：2
                ThreadA 添加了元素：3
                ThreadA 添加了元素：4
                已经发出通知，相应的线程被停止，停止时元素数量为： 4
                ThreadA 添加了元素：5
                ThreadA 添加了元素：6
                ThreadA 添加了元素：7
                ThreadA 添加了元素：8
                ThreadA 添加了元素：9
                ThreadA 退出对象锁
                ThreadA 退出
                java.lang.InterruptedException
                    at practice.ThreadB.run(ThreadB.java:51)
                    at java.base/java.lang.Thread.run(Thread.java:844)
                ThreadB 退出对象锁
                ThreadB 退出

#### 3) 线程状态切换
> - 线程进入Runnable的原因
>> - 调用sleep()方法经过一定时间后返回
>> - 线程调用的阻塞IO已经返回，阻塞方法执行完毕
>> - 线程获得了试图同步的监视器
>> - 线程正在等待某个消息，其他线程发出了通知
>> - 处于挂起的线程调用了resume()方法  
> - 线程出现阻塞的原因
>> - 调用sleep()方法
>> - 线程调用的阻塞IO
>> - 线程试图获得了试图同步的监视器，但是被其他线程所占有
>> - 线程正在等待某个消息
>> - 使用suspend()被挂起
> - 每个所对象都有两个队列，一个是就绪队列，另一个是阻塞队列
>>>>>> ![图1-5 线程状态切换.jpg](https://github.com/hblvsjtu/JavaMultiThreadProgramming_Study/blob/master/picture/%E5%9B%BE1-4%20%E5%8F%98%E9%87%8F%E5%9C%A8%E5%86%85%E5%AD%98%E4%B8%AD%E7%9A%84%E5%B7%A5%E4%BD%9C%E8%BF%87%E7%A8%8B.jpg?raw=true)

            
<h3 id='3.2'>3.2 通过管道进行线程间通讯</h3>
            
#### 1) 介绍
> - 管道流是一种特殊的流，用于不同线程间直接传送数据
> - 无需借助于临时文件之类的东西
#### 2) 字节流
> - PipedInputStream & PipedOutputStream
> - 由于线程B一开始没有数据写入，所以遇到输入输出的语句时线程被阻塞，直到有数据写入
> - 每次输入对应一次输出
                
                package practice;

                import java.io.IOException;
                import java.io.PipedOutputStream;
                import java.util.ArrayList;
                import java.util.List;

                /**
                 * @author LvHongbin
                 * ThreadA类
                 */
                public class ThreadA implements Runnable {

                    private List<Integer> myList;
                    private PipedOutputStream out;

                    /**
                     * @param myList
                     */
                    public ThreadA() {
                        super();
                        this.myList = new ArrayList<Integer>();
                        this.out = new PipedOutputStream();
                    }
                    

                    /**
                     * @return the list
                     */
                    public List<Integer> getMyList() {
                        return myList;
                    }


                    /**
                     * @param list the list to set
                     */
                    public void setMyList(List<Integer> myList) {
                        this.myList = myList;
                    }

                    
                    /**
                     * @return the out
                     */
                    public PipedOutputStream getOut() {
                        return out;
                    }


                    /**
                     * @param out the out to set
                     */
                    public void setOut(PipedOutputStream out) {
                        this.out = out;
                    }


                    @Override
                    public void run() {
                        System.out.println(Thread.currentThread().getName() + " 开启");
                        for(int i=0; i<10; i++) {
                            this.myList.add(i);
                            System.out.println(Thread.currentThread().getName() + " 添加了元素：" + i);
                            if (this.myList.size() == 5) {
                                System.out.println(Thread.currentThread().getName() + " 进入判断的第一行");
                                try {
                                    this.out.write(5);
                                } catch (IOException e) {
                                    e.printStackTrace();
                                }
                                System.out.println("已经发出通知，相应的线程被停止，停止时元素数量为： " + i);
                            }
                        }
                        System.out.println(Thread.currentThread().getName() + " 退出循环");
                        System.out.println(Thread.currentThread().getName() + " 退出");
                    }
                }

                package practice;

                import java.io.IOException;
                import java.io.PipedInputStream;

                /**
                 * @author LvHongbin
                 * ThreadB类
                 */
                public class ThreadB implements Runnable{


                    private PipedInputStream in;

                    public ThreadB() {
                        super();
                        this.in = new PipedInputStream();
                    }

                    /**
                     * @return the in
                     */
                    public PipedInputStream getIn() {
                        return in;
                    }


                    /**
                     * @param in the in to set
                     */
                    public void setIn(PipedInputStream in) {
                        this.in = in;
                    }


                    @Override
                    public void run() {
                        System.out.println(Thread.currentThread().getName() + " 开启 ");
                        try {
                            if (in.read() != -1) {
                                System.out.println(Thread.currentThread().getName() + " 进入判断的第一行");
                                try {
                                    Thread.currentThread().interrupt();
                                    throw new InterruptedException();
                                }catch(InterruptedException e) {
                                    e.printStackTrace();
                                }
                                System.out.println(Thread.currentThread().getName() + " 退出判断");
                            }
                        } catch (IOException e) {
                            e.printStackTrace();
                        }
                        System.out.println(Thread.currentThread().getName() + " 退出");
                    }
                }

                // 执行类
                ThreadA a = new ThreadA();
                ThreadB b = new ThreadB();
                a.getOut().connect(b.getIn());
                Thread ta = new Thread(a, "ThreadA");
                Thread tb = new Thread(b, "ThreadB");
                tb.start();
                ta.start();

                // 控制台
                ThreadA 添加了元素：0
                ThreadA 添加了元素：1
                ThreadA 添加了元素：2
                ThreadA 添加了元素：3
                ThreadA 添加了元素：4
                ThreadA 进入判断的第一行
                已经发出通知，相应的线程被停止，停止时元素数量为： 4
                ThreadA 添加了元素：5
                ThreadA 添加了元素：6
                ThreadA 添加了元素：7
                ThreadA 添加了元素：8
                ThreadA 添加了元素：9
                ThreadA 退出循环
                ThreadA 退出
                ThreadB 进入判断的第一行
                java.lang.InterruptedException
                    at practice.ThreadB.run(ThreadB.java:52)
                    at java.base/java.lang.Thread.run(Thread.java:844)
                ThreadB 退出判断
                ThreadB 退出

                //只输出一次和接收一次
                package practice;

                import java.io.IOException;
                import java.io.PipedOutputStream;
                import java.util.ArrayList;
                import java.util.List;

                /**
                 * @author LvHongbin
                 * ThreadA类
                 */
                public class ThreadA implements Runnable {

                    private List<Integer> myList;
                    private PipedOutputStream out;

                    /**
                     * @param myList
                     */
                    public ThreadA() {
                        super();
                        this.myList = new ArrayList<Integer>();
                        this.out = new PipedOutputStream();
                    }
                    

                    /**
                     * @return the list
                     */
                    public List<Integer> getMyList() {
                        return myList;
                    }


                    /**
                     * @param list the list to set
                     */
                    public void setMyList(List<Integer> myList) {
                        this.myList = myList;
                    }

                    
                    /**
                     * @return the out
                     */
                    public PipedOutputStream getOut() {
                        return out;
                    }


                    /**
                     * @param out the out to set
                     */
                    public void setOut(PipedOutputStream out) {
                        this.out = out;
                    }


                    @Override
                    public void run() {
                        System.out.println(Thread.currentThread().getName() + " 开启");
                        for(int i=0; i<10; i++) {
                            this.myList.add(i);
                            System.out.println(Thread.currentThread().getName() + " 添加了元素：" + i);
                            try {
                                this.out.write(i);
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }
                        try {
                            this.out.close();
                            System.out.println(Thread.currentThread().getName() + " 退出循环，关闭输出流");
                        } catch (IOException e) {
                            // TODO Auto-generated catch block
                            e.printStackTrace();
                        }
                        System.out.println(Thread.currentThread().getName() + " 退出");
                    }
                }

                package practice;

                import java.io.IOException;
                import java.io.PipedInputStream;

                /**
                 * @author LvHongbin
                 * ThreadB类
                 */
                public class ThreadB implements Runnable{


                    private PipedInputStream in;

                    public ThreadB() {
                        super();
                        this.in = new PipedInputStream();
                    }

                    /**
                     * @return the in
                     */
                    public PipedInputStream getIn() {
                        return in;
                    }


                    /**
                     * @param in the in to set
                     */
                    public void setIn(PipedInputStream in) {
                        this.in = in;
                    }


                    @Override
                    public void run() {
                        System.out.println(Thread.currentThread().getName() + " 开启 ");
                        int num = 0;
                        try {
                            num = in.read();
                            System.out.println(Thread.currentThread().getName() + " 添加了元素：" + num);
                            if (num == 5) {
                                System.out.println(Thread.currentThread().getName() + " 进入判断的第一行");
                                try {
                                    Thread.currentThread().interrupt();
                                    throw new InterruptedException();
                                }catch(InterruptedException e) {
                                    e.printStackTrace();
                                }
                                System.out.println(Thread.currentThread().getName() + " 退出判断");
                            }
                        } catch (IOException e) {
                            e.printStackTrace();
                            System.out.println("已经发出通知，相应的线程被停止，停止时元素数量为： " + num);
                        }
                        System.out.println(Thread.currentThread().getName() + " 退出");
                    }
                }

                //执行类
                ThreadA a = new ThreadA();
                ThreadB b = new ThreadB();
                a.getOut().connect(b.getIn());
                Thread ta = new Thread(a, "ThreadA");
                Thread tb = new Thread(b, "ThreadB");
                tb.start();
                ta.start();

                // 控制台
                ThreadB 开启 
                ThreadA 开启
                ThreadA 添加了元素：0
                ThreadA 添加了元素：1
                ThreadA 添加了元素：2
                ThreadA 添加了元素：3
                ThreadA 添加了元素：4
                ThreadA 添加了元素：5
                ThreadA 添加了元素：6
                ThreadA 添加了元素：7
                ThreadA 添加了元素：8
                ThreadA 添加了元素：9
                ThreadA 退出循环，关闭输出流
                ThreadB 添加了元素：0
                ThreadA 退出
                ThreadB 退出
#### 3) 字符流
> - PipedReader & PipedWriter
> - 
<h3 id='3.3'>3.3 方法join 类ThreadLocal和类inheritableThreadLocal</h3>
            
#### 1) 方法join
> - 对被作用的线程继续运行run()方法，而其他线程则无限期延长，直至被作用的方法已经执行完毕
> - 有种同步的感觉，跟synchronied不同的是，join实在内部使用wait()方法，继而释放锁，其他线程可以执行其同步的方法。而synchronied则是“对象监视器”原理
> - join(long)则内置wait(long)
#### 2) 类ThreadLocal
> - 一般来讲类的静态实例域就是全局变量，放在堆中，而类ThreadLocal则是为每个线程创建绑定自己的值
> - 初值为null，可以覆写该类的initialValue来改写初值
                
                public static ThreadLocal t1 = new ThreadLocal();
                t1.set(...);
                t1.get();
#### 3) 类inheritableThreadLocal
> - 可以在子线程中获取父线程中的值

<h3 id='3.4'>3.4 类inheritableThreadLocal的使用</h3>
            
#### 1) 对象锁
> - 