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
![cluster.jpg](cluster.jpg)
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
## 4、本地调用 ##

## 5、本地伪装 ##

## 6、本地存根 ##

## 7、泛化调用 ##


