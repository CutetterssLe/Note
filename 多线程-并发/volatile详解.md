- [1、volatile作用详解](#1volatile作用详解)
  - [1.1 防重排序（禁止指令重排）](#11-防重排序禁止指令重排)
  - [1.2 实现可见性](#12-实现可见性)
  - [1.3 保证单次读写的原子性](#13-保证单次读写的原子性)
- [2、volatile实现原理](#2volatile实现原理)
  - [2.1 volatile 可见性实现](#21-volatile-可见性实现)
  - [2.2 volatile 有序性实现](#22-volatile-有序性实现)
- [3、volatile使用场景](#3volatile使用场景)

# 1、volatile作用详解
## 1.1 防重排序（禁止指令重排）
我们从一个最经典的例子来分析重排序问题。大家应该都很熟悉单例模式的实现，而在并发环境下的单例实现方式，我们通常可以采用双重检查加锁(DCL)的方式来实现。其源码如下：

    public class Singleton {
        public static volatile Singleton singleton;
        /**
        * 构造函数私有，禁止外部实例化
        */
        private Singleton() {};
        public static Singleton getInstance() {
            if (singleton == null) {
                synchronized (singleton.class) {
                    if (singleton == null) {
                        singleton = new Singleton();
                    }
                }
            }
            return singleton;
        }
    }
实例化一个对象分为3个步骤：
- 分配内存空间。
- 初始化对象。
- 将内存空间的地址赋值给对应的引用。

但是由于操作系统可以对指令进行重排序，所以上面的过程可能并不是顺序执行。
## 1.2 实现可见性
可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——线程工作内存。volatile关键字能有效的解决这个问题。
## 1.3 保证单次读写的原子性
- 为什么i++不保证原子性？
  - 代码：

        public void ts() {
            int i = 0;
            i++;
            System.out.println(i);
        }
  - 字节码：

        0 iconst_0 //将 int 型 0 推送至栈顶
        1 istore_1 //将栈顶 int 型数值存入第二个本地变量
        2 iinc 1 by 1 //将指定 int 型变量增加指定值（i++,
        5 getstatic #2 <java/lang/System.out : Ljava/io/PrintStream;>
        8 iload_1 //将第二个 int 型本地变量推送至栈顶
        9 invokevirtual #3 <java/io/PrintStream.println : (I)V>
        12 return

从字节码层面看，i++分为了好几步，是线程不安全的
# 2、volatile实现原理
volatile 变量的内存可见性是基于内存屏障(Memory Barrier)实现的。
## 2.1 volatile 可见性实现
- 内存屏障，又称内存栅栏，是一个 CPU 指令。
- 在程序运行时，为了提高执行性能，编译器和处理器会对指令进行重排序，JMM 为了保证在不同的编译器和 CPU 上有相同的结果，通过插入特定类型的内存屏障来禁止+ 特定类型的编译器重排序和处理器重排序，插入一条内存屏障会告诉编译器和 CPU：不管什么指令都不能和这条 Memory Barrier 指令重排序。 
  - 如果对声明了 volatile 的变量进行写操作，JVM 就会向处理器发送一条 lock 前缀的指令，将这个变量所在缓存行的数据写回到系统内存。
  
        在 Pentium 和早期的 IA-32 处理器中，lock 前缀会使处理器执行当前指令时产生一个 LOCK# 信号，会对总线进行锁定，其它 CPU 对内存的读写请求都会被阻塞，直到锁释放。后来的处理器，加锁操作是由高速缓存锁代替总线锁来处理。因为锁总线的开销比较大，锁总线期间其他 CPU 没法访问内存。这种场景多缓存的数据一致通过缓存一致性协议(MESI)来保证。
 
  - 为了保证各个处理器的缓存是一致的，实现了缓存一致性协议(MESI)，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。 

        缓存是分段(line)的，一个段对应一块存储空间，称之为缓存行，它是 CPU 缓存中可分配的最小存储单元，大小 32 字节、64 字节、128 字节不等，这与 CPU 架构有关，通常来说是 64 字节。
        LOCK# 因为锁总线效率太低，因此使用了多组缓存。
        为了使其行为看起来如同一组缓存那样。因而设计了 缓存一致性协议。
        缓存一致性协议有多种，但是日常处理的大多数计算机设备都属于 " 嗅探(snooping)" 协议。

  - volatile 变量通过这样的机制就使得每个线程都能获得该变量的最新值。
## 2.2 volatile 有序性实现
- happens-before原则

    happens-before 规则中有一条是 volatile 变量规则：对一个 volatile 域的写，happens-before 于任意后续对这个 volatile 域的读。
- volatile 禁止重排序
  - 为了性能优化，JMM 在不改变正确语义的前提下，会允许编译器和处理器对指令序列进行重排序。JMM 提供了内存屏障阻止这种重排序。
  - Java 编译器会在生成指令系列时在适当的位置会插入内存屏障指令来禁止特定类型的处理器重排序。
    - 在每个 volatile 写操作的前面插入一个 StoreStore 屏障。 
    - 在每个 volatile 写操作的后面插入一个 StoreLoad 屏障。 
    - 在每个 volatile 读操作的后面插入一个 LoadLoad 屏障。 
    - 在每个 volatile 读操作的后面插入一个 LoadStore 屏障。
# 3、volatile使用场景
- 双重检查锁，单例对象
- java.util.concurrent






