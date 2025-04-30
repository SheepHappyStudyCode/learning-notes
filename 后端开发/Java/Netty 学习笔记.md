# Netty 学习笔记

## 参考文章

1. [肝了一个月的Netty知识点（上）高能预警，本文是我一个月前就开始写的，所以内容会非常长，当然也非常硬核，dubbo源码 - 掘金](https://juejin.cn/post/6921858121774137352?searchId=202503282238005CC857A3361EB7929FC3)
1. [超详细Netty入门，看这篇就够了！-阿里云开发者社区](https://developer.aliyun.com/article/769587)

## Netty 是什么

Netty 是一个 **NIO 客户端服务器框架**，可快速轻松地开发网络应用程序，例如协议服务器和客户端。它极大地简化和简化了**网络编程**。

“快速简便”并不意味着最终的应用程序将遭受可维护性或性能问题的困扰。Netty 经过精心设计，结合了许多协议（例如FTP，SMTP，HTTP 以及各种基于二进制和文本的旧式协议）的实施经验。结果，Netty 成功地找到了一种无需妥协即可轻松实现开发，性能，稳定性和灵活性的方法。

## 架构图

![image.png](https://gitee.com/SheeepHappy/blog-pic/raw/master/20250329102142722.png)



## Netty 为什么适合搭建高性能服务器

1. 使用 Reactor 线程模型，Boss 线程组用来处理建连事件，Worker 线程组用来处理 channel 上的 IO 事件，异步非阻塞，效率高。
2. 使用零拷贝技术，减少文件、bytebuf 的拷贝次数