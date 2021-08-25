# 1、方法区
- jdk7以前：永久代
- jdk7及以后：元空间
# 2、虚拟机栈
- JVM中有多少个虚拟机栈？
  - 一个线程对应一个
- 一个栈有多少栈帧？
  - 方法调用次数个

java -XX:PrintFlagsFinal -version | grep ThreadStackSize 查看默认栈大小  1024k

轻量级锁借助栈帧实现

栈帧由以下几部分组成：
  - 局部变量表
    - 存放局部变量的地方
  - 操作数栈
    - bipush
    - iadd
    - dup
    - ···
    - 利用操作数栈把值放进局部变量表
  - 动态链接
    - 方法在对应JVM对象(instanceKlass)在元空间的内存地址
    - VTABLE，存储形式类似List< MethodObj > 
  - 返回地址
# 3、程序计数器（字节码索引）
