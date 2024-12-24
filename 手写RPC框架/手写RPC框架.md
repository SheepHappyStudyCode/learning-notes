# 手写RPC框架



## 什么是RPC框架

rpc，即**远程过程调用**，主要用于后端服务器之间的通信，能够帮助开发者**像调用本地方法一样调用远程的方法**。



## RPC 框架做了什么事情

### 消费者端

创建了一个**代理对象**，该对象代替用户发送请求，接收响应。

### 服务提供者端

有一个**服务器**用于接收请求，接受的请求通过**请求处理器**，请求处理器通过映射和反射机制生成对应的实现类和方法，返回给消费者相应的数据。



## RPC 框架的架构图

![image-20240818160936315](img\image-20240818160936315.png)

虚线部分就是 RPC 做的工作



## 双检索单例模式

```java
public class Singleton {
    // 使用volatile关键字确保多线程环境下的可见性和有序性
    private static volatile Singleton instance;

    // 私有构造函数，防止外部实例化
    private Singleton() {}

    // 提供一个全局访问点
    public static Singleton getInstance() {
        // 第一次检查，不加锁
        if (instance == null) {
            // 同步块，确保只有一个线程可以进入
            synchronized (Singleton.class) {
                // 第二次检查，加锁
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

```

### 为什么 Singleton.class 可以实现同步

当JVM加载一个类时，它会创建一个与之关联的Class对象。这个Class对象包含了类的元数据，如类的名称、字段、方法等。**每个类只有一个Class对象**，因此，无论多少个线程访问这个类，它们获取的都是同一个Class对象。

### 关键词解释

1. **volatile关键字**：`volatile`关键字确保`instance`变量的修改对所有线程可见，防止指令重排序。
2. **双重检查**：第一次检查是为了避免不必要的同步，第二次检查是为了确保在同步块内只有一个线程可以创建实例。
3. **同步块**：使用`synchronized`关键字确保在多线程环境下只有一个线程可以进入同步块。