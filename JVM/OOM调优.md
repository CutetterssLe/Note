# 1、TLAB
TLAB，即Thread Local Alloc Buffer，是线程的一块私有内存。关于TLAB，你是不是有很多问号？比如：为什么要引入TLAB技术、TLAB解决了什么问题、TLAB占多大内存、什么样的对象能在TLAB中分配……
## 1.1、为什么引入TLAB
***
日常工作中，创建对象是一个很频繁的操作。JVM是如何创建对象的呢？先从堆区分配一块对象大小的内存（栈上分配后面写文章讲），然后……在多线程环境下，为了保证操作的原子性，势必需要借助一些策略来控制。
***
目前主流的内存分配策略有两种：**指针碰撞、空闲列表**。虽然这两种策略能保证并发环境下分配内存的原子性，但是因为底层使用了CAS循环碰撞、借助其他数据结构记录，势必带来了额外的性能开销。那就没有一种更高效的机制吗？有，就是TLAB，是在JDK1.6引入的一项技术。
## 1.2、TLAB是什么
TLAB是线程的一块私有内存，即这块buffer只给当前线程使用。只给当前线程使用的意思是只有当前线程可以向这块内存写入数据，但是写入的数据依然是线程共享的。即TLAB只有当前线程可写，所有线程可读。
***
JDK6以后TLAB默认是开启的，可以通过参数-XX:+/-UseTLAB设置开启或关闭，建议开启。
***
TLAB是从Eden区分出来的，属于堆区内存。开启TLAB有什么好处呢？好处就是在多线程环境下申请内存就不存在竞争，直接在当前线程的buffer上分配即可。这里就涉及到问题：什么样的对象可以在TLAB上分配？
***
- TLAB结构：

        class ThreadLocalAllocBuffer: publicCHeapObj<mtThread>{
            friend class VMStructs;
        private:
        HeapWord* _start;                              // address of TLAB
        HeapWord* _top;                                // address after last allocation
        HeapWord* _pf_top;                             // allocation prefetch watermark
        HeapWord* _end;                                // allocation end (excluding alignment_reserve)
        size_t    _desired_size;                       // desired size   (including alignment_reserve)
        size_t    _refill_waste_limit;                 // hold onto tlab if free() is larger than this

        static unsigned _target_refills;               // expected number of refills between GCs

        unsigned  _number_of_refills;
        unsigned  _fast_refill_waste;
        unsigned  _slow_refill_waste;
        unsigned  _gc_waste;
        unsigned  _slow_allocations;

        AdaptiveWeightedAverage _allocation_fraction;
        ……
- TLAB何时初始化
TLAB是在创建Java线程的时候初始化的

        // The first routine called by a new Java thread
        void JavaThread::run() {
        // initialize thread-local alloc buffer related fields
        this->initialize_tlab();
        ……  

        // Thread-Local Allocation Buffer (TLAB) support
        ThreadLocalAllocBuffer& tlab()                 { return _tlab; }
        void initialize_tlab() {
            if (UseTLAB) {
            tlab().initialize();
            }
        }    
# 2、PLAB
在老年代中分配的一块线程私有的内存，即promotion LAB。
