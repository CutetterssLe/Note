- [Apache Dubbo 是一款高性能、轻量级的开源服务框架](#apache-dubbo-是一款高性能轻量级的开源服务框架)
  - [1、启动时检查 ##](#1启动时检查-)
  - [2、集群容错](#2集群容错)
    - [集群容错模式](#集群容错模式)
    - [集群模式配置](#集群模式配置)
  - [3、多协议](#3多协议)
  - [4、负载均衡](#4负载均衡)
  - [4、服务超时](#4服务超时)
  - [5、本地伪装](#5本地伪装)
  - [6、本地存根](#6本地存根)
  - [7、泛化调用](#7泛化调用)
  - [8、参数回调](#8参数回调)
  - [9、SPI机制](#9spi机制)
# Apache Dubbo 是一款高性能、轻量级的开源服务框架 #
## 1、启动时检查 ## 
    Dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，以便上线时，能及早发现问题，默认 check="true"。
- spring配置文件方式
    - 关闭某个服务启动时检查：<dubbo:reference interface="com.foo.BarService" check="false" />
    - 关闭所有服务启动检查：<dubbo:consumer check="false" />
    - 关闭注册中心启动检查：<dubbo:registry check="false" />
- dubbo.properties方式
    - dubbo.reference.com.foo.BarService.check=false
    - dubbo.consumer.check=false
    - dubbo.registry.check=false
- 通过-D参数
    - java -Ddubbo.reference.com.foo.BarService.check=false
    - java -Ddubbo.consumer.check=false 
    - java -Ddubbo.registry.check=false
---
## 2、集群容错 ##
    在集群调用失败时，Dubbo 提供了多种容错方案，缺省为 failover 重试。
![cluster.jpg](../image/cluster.jpg)
- 这里的 Invoker 是 Provider 的一个可调用 Service 的抽象，Invoker 封装了 Provider 地址及 Service 接口信息
- Directory 代表多个 Invoker，可以把它看成 List<Invoker> ，但与 List 不同的是，它的值可能是动态变化的，比如注册中心推送变更
- Cluster 将 Directory 中的多个 Invoker 伪装成一个 Invoker，对上层透明，伪装过程包含了容错逻辑，调用失败后，重试另一个
- Router 负责从多个 Invoker 中按路由规则选出子集，比如读写分离，应用隔离等
- LoadBalance 负责从多个 Invoker 中选出具体的一个用于本次调用，选的过程包含了负载均衡算法，调用失败后，需要重选
### 集群容错模式 ###
- Failover Cluster
  
    失败自动切换，当出现失败，重试其它服务器。通常用于读操作，但重试会带来更长延迟。可通过 retries="2" 来设置重试次数(不含第一次)。
    ***
    <dubbo:service retries="2" />
    ***
    <dubbo:reference retries="2" />
    ***
    < dubbo:reference >
    
    <dubbo:method name="findFoo" retries="2" />

    </dubbo:reference>
    ***
- Failfast Cluster 
  
    快速失败，只发起一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增记录。
- Failsafe Cluster 
  
    失败安全，出现异常时，直接忽略。通常用于写入审计日志等操作。
- Failback Cluster 

    失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知操作。
- Forking Cluster 

    并行调用多个服务器，只要一个成功即返回。通常用于实时性要求较高的读操作，但需要浪费更多服务资源。可通过 forks="2"(集群数量) 来设置最大并行数。
- Broadcast Cluster 

    广播调用所有提供者，逐个调用，任意一台报错则报错。通常用于通知所有提供者更新缓存或日志等本地资源信息。

    @reference(cluster = "broadcast", parameters = {"broadcast.fail.percent", "20"})

    说明：当20%个节点失败后，BroadcastClusterInvoker 将不再调用其他节点，直接抛出异常。取值范围0 - 100，此参数在2.7.10及以上版本生效

### 集群模式配置 ###

    <dubbo:service cluster="failsafe" />
    或者
    <dubbo:reference cluster="failsafe" />
## 3、多协议 ##

    Dubbo 允许配置多协议，在不同服务上支持不同协议或者同一服务上同时支持多种协议。

    <!-- 多协议配置 -->

    <dubbo:protocol name="dubbo" port="20880" />

    <dubbo:protocol name="rmi" port="1099" />

    <!-- 使用dubbo协议暴露服务 -->
    
    <dubbo:service interface="com.alibaba.hello.api.HelloService" version="1.0.0" ref="helloService" protocol="dubbo" />

    <!-- 使用rmi协议暴露服务 -->

    <dubbo:service interface="com.alibaba.hello.api.DemoService" version="1.0.0" ref="demoService" protocol="rmi" /> 

    <!-- 使用多个协议暴露服务 -->

    <dubbo:service id="helloService" interface="com.alibaba.hello.api.HelloService" version="1.0.0" protocol="dubbo,hessian" />
## 4、负载均衡 ##

    在集群负载均衡时，Dubbo 提供了多种均衡策略，缺省为 random 随机调用。
  - Random LoadBalance ： 随机

  - RoundRobin LoadBalance ： 轮询
  - LeastActive LoadBalance ： 最小活跃度

        由消费端去统计，active属性，调用某个服务之后 active + 1， 返回结果之后 active - 1
    
  - ConsistentHash LoadBalance ： 一致性Hash

        根据参数做一致性Hash算法
  - 服务端服务级别配置

        <dubbo:service interface="..." loadbalance="roundrobin" />
  - 客户端服务级别配置
  
        <dubbo:reference interface="..." loadbalance="consistenthash" />
  - 服务端方法级别配置

        <dubbo:service interface="...">
            <dubbo:method name="..." loadbalance="roundrobin"/>
        </dubbo:service>
  - 客户端服务级别配置
  
        <dubbo:reference interface="...">
            <dubbo:method name="..." loadbalance="roundrobin"/>
        </dubbo:reference>
## 4、服务超时 ##

    客户端配置timeout = 3000， 服务端配置timeout = 6000， 服务执行5秒钟，客户端报timeoutexception，服务端无报错
## 5、本地伪装 ##

## 6、本地存根 ##

## 7、泛化调用 ##

## 8、参数回调 ##

## 9、SPI机制 ##
-   在需要使用SPI的Service上面加注解@SPI，并在resources/META-INF/dubbo新建接口全限定名文件，并配置key = 实现类全限定名，使用如下方法去获取：

        ExtensionLoader<Person> extensionLoader = ExtensionLoader.getExtensionLoader(Person.class);
        Person person = extensionLoader.getExtension("key");
        System.out.println(person);
-   提供类似IOC、AOP功能
  
        public interface Car {
            public String getName();
        }

        public class RedCar implements Car{
            @Override
            public String getName() {
                return "red";
            }
        }

        public class CarWrapper implements Car{

            private Car car;

            public CarWrapper(Car car) {
                this.car = car;
            }
            @Override
            public String getName() {
                return "wrapper";
            }
        }

    配置文件：

        black = com.xx.BlackCar
        com.xx.CarWrapper

    通过extensionLoader.getExtension("key")这个时候拿出来的对象其实是Wrapper对象，我们可以对Car做一个包装，dosomething
-   源码解析

    ExtensionLoader<Person> extensionLoader = ExtensionLoader.getExtensionLoader(Person.class);


        public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
                // 前面都是一些校验
            if (type == null)
                throw new IllegalArgumentException("Extension type == null");
            if (!type.isInterface()) {
                throw new IllegalArgumentException("Extension type(" + type + ") is not interface!");
            }
            if (!withExtensionAnnotation(type)) {
                throw new IllegalArgumentException("Extension type(" + type +
                        ") is not extension, because WITHOUT @" + SPI.class.getSimpleName() + " Annotation!");
            }
            //EXTENSION_LOADERS  
            private static final ConcurrentMap<Class<?>, ExtensionLoader<?>> EXTENSION_LOADERS = new ConcurrentHashMap<Class<?ExtensionLoader<?>>();
            //从map中获取一个type类型的
            ExtensionLoader<T> loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
            if (loader == null) {
                EXTENSION_LOADERS.putIfAbsent(type, new ExtensionLoader<T>(type));
                loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
            }
            return loader;
        }
    Person person = extensionLoader.getExtension("key");

        public T getExtension(String name) {
            if (name == null || name.length() == 0)
                throw new IllegalArgumentException("Extension name == null");
            //如果name = true，就会获取默认实现，就是@SPI注解后面的参数，例如@SPI("black")，默认获取blackCar
            if ("true".equals(name)) {
                return getDefaultExtension();
            }
            //Holder主要是解决并发问题
            Holder<Object> holder = cachedInstances.get(name);
            if (holder == null) {
                cachedInstances.putIfAbsent(name, new Holder<Object>());
                holder = cachedInstances.get(name);
            }
            //如果有2个线程同时进来获取，只有一个线程会创建
            Object instance = holder.get();
            if (instance == null) {
                //使用holder作为锁
                synchronized (holder) {
                    instance = holder.get();
                    if (instance == null) {
                        //创建扩展点实例对象，步骤：
                        1、直接从缓存中获取对应class实例
                        2、如果获取不到，使用双重检查锁，加载解析配置类
                            getExtensionClasses() -> loadExtensionClasses()，这一步会把默认扩展类进行缓存，方便后面使用
                        3、进行依赖注入IOC injectExtension(instance);
                        4、AOP Set<Class<?>> wrapperClasses = cachedWrapperClasses;
                        instance = createExtension(name);
                        holder.set(instance);
                    }
                }
            }
            return (T) instance;
        }
如果在指定的别的类里面需要注入Car，需要生成一个Car代理对象，给Car的getName方法上面加@Adaptive注解，参数需要有一个URL对象（或者参数对象有个方法叫getUrl），Dubbo会拼装出一个代理类，如果Car接口内有方法没加注解，生成的代理类调用这个方法会直接抛异常UnsupportedOperationException，方法不是Adaptive方法。



