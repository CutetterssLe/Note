# 典型的多路复用IO实现
| IO模型 | 相对性能 | 关键思路 | 操作系统 | JAVA支持情况 |
| --- | --- | --- | --- | --- |
| select | 较高 | Reactor | windows/Linux | 支持,Reactor模式(反应器设计模式)。Linux操作系统的 kernels 2.4内核版本之前，默认使用select；而目前windows下对同步IO的支持，都是select模型 |
| poll | 较高 | Reactor | Linux | Linux下的JAVA NIO框架，Linux kernels 2.6内核版本之前使用poll进行支持。也是使用的Reactor模式 |
| epoll | 高 | Reactor/Proactor | Linux | Linux kernels 2.6内核版本及以后使用epoll进行支持；Linux kernels 2.6内核版本之前使用poll进行支持；另外一定注意，由于Linux下没有Windows下的IOCP技术提供真正的 异步IO 支持，所以Linux下使用epoll模拟异步IO |
| kqueue | 高 | Proactor | Linux | 目前JAVA的版本不支持 |
***

多路复用IO技术最适用的是“高并发”场景，所谓高并发是指1毫秒内至少同时有上千个连接请求准备好。其他情况下多路复用IO技术发挥不出来它的优势。另一方面，使用JAVA NIO进行功能实现，相对于传统的Socket套接字实现要复杂一些，所以实际应用中，需要根据自己的业务需求进行技术选择
