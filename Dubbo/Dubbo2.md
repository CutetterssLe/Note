# Apache Dubbo 是一款高性能、轻量级的开源服务框架 #
## 1、启动时检查 ## 
    Dubbo 缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止 Spring 初始化完成，以便上线时，能及早发现问题，默认 check="true"。
- spring配置文件方式：
    - 关闭某个服务启动时检查：<dubbo:reference interface="com.foo.BarService" check="false" />
    - 关闭所有服务启动检查：<dubbo:consumer check="false" />
    - 关闭注册中心启动检查：<dubbo:registry check="false" />
- dubbo.properties方式：
    - dubbo.reference.com.foo.BarService.check=false
    - dubbo.consumer.check=false
    - dubbo.registry.check=false
- 通过-D参数：
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



