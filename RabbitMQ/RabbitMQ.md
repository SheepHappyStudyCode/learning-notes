# RabbitMQ
## 1 MQ基本概念
### 1.1 MQ概述
消息队列，是在消息的传输过程中保存信息的容器，多用于分布式系统通信。
### 1.2 MQ的优势和劣势
1. 优势：
	- **应用解耦**：提高系统容错性和可维护性
	- **异步提速**：提升用户体验和系统吞吐量
	- **削峰填谷**：提高系统稳定性

2. 劣势：
	- 系统可用性降低：一旦MQ宕机就不可用
	- 系统复杂性增加
	- 一致性问题

3. MQ使用场景：
	- 消费者不需要从生产者处得到反馈。引入消息队列之前的直接调用，其接口的返回值应该为空。
	- 允许短暂的不一致性。
	- MQ带来的提升比管理MQ的成本要大。
### 1.3 常见的MQ产品

![image-20240615172702403](img\image-20240615172702403.png)

### 1.4 RabbitMQ简介
1. **AMQP协议**：高级消息队列协议。是一个网络协议，是应用层协议的一个开放标准，为面向消息的中间件设计。
2. RabbitMQ**基础架构**：
![image-20240615173900766](img\image-20240615173900766.png)
3. RabbitMQ提供了**6种工作模式**。
4. JMS：Java消息服务应用程序接口，是JavaEE的技术规范之一。

## 2 RabbitMQ的工作模式
### 2.1 Work queues工作队列模式
![image-20240618212058854](img\image-20240618212058854.png)

应用场景：适用于任务过重或任务较多的情况。

### 2.2 Pub/Sub 订阅模式
![image-20240618213045090](img\image-20240618213045090.png)

1. 交换机（X）：接受生产者的消息，再将消息分发出去。
3. **交换机的模式**：
	- direct：定向，消息精准匹配路由键到队列
	- fanout：广播。发送消息到每一个与之绑定的队列。
	- topic：通配符的方式。
	- headers：通过消息的头部信息进行路由，而不是路由键。绑定的队列会根据头部信息中的键值对进行路由

### 2.3 Routing 路由模式
1. 模式说明：
	- 交换机类型选择 fanout
	- 队列和交换机的绑定不是任意的，而是要指定一个 RoutingKey
	- 消息的发送方在向Exchange发送消息时，也必须指定消息的RoutingKey
	- Exchange根据消息的RoutingKey进行判断，只有**队列的RoutingKey与消息的RoutingKey一致**，才会接受到消息。
	
	
	![image-20240620082447657](img\image-20240620082447657.png)

### 2.4 Topics 通配符模式
![image-20240620090002774](img\image-20240620090002774.png)

## 3 RabbitMQ的高级特性
### 3.1 消息持久化

1. 消费者发送消息默认是持久化的（spring2中）
2. 消息队列最好设置为持久化：不持久化的话，重启 mq 队列就没了
3. 只有**消息和队列同时设置为持久化**才会生效

### 3.2 Consumer ACK
1. ack指 acknowledge 。表示消费端收到消息后的确认方式。
2. 三种确认方式：
	- 自动确认：acknowledge="none"
	- 手动确认：acknowledge="manual"
	- 根据异常情况确认（比较麻烦）
### 3.3 消费端限流

>  消费端的限流（Rate Limiting）是控制消费者处理消息的速度，防止消费者处理过快或过慢导致的系统过载、资源浪费、或者消息堆积。

#### 解决方法

1. **QoS** 配置，限制每次消费的消息数量。
   - 可以通过设置 `prefetch`来实现

2. **人为延时**，控制消息消费的间隔。

3. **令牌桶算法**，精确控制消费速率。可以使用第三方库（如 Guava）来实现令牌桶限流：



配置文件：

```yml
spring:
  rabbitmq:
    host: 39.105.36.211
    username: guest
    password: yh520js666
    virtual-host: /
    port: 5672
    # rabbitmq 消费者监听配置
    listener:
      simple:
        concurrency: 5 # 消费端的监听个数(即 @RabbitListener开启几个线程去处理数据。)
        max-concurrency: 10 #最大并发
        acknowledge-mode: manual # 签收模式
        prefetch: 1 # 消费端限流 ，每个线程最多取一个数据
```

### 3.4 TTL

> 在 RabbitMQ 中，消息 TTL 分为队列级 TTL 和 消息级 TTL

#### 队列级 TTL

```JAVA
Map<String, Object> arguments = new HashMap<>();
arguments.put("x-message-ttl", 60000);  // 消息的 TTL 为 60 秒
arguments.put("x-dead-letter-exchange", "dlx-exchange"); // 设置死信交换机
arguments.put("x-dead-letter-routing-key", "dlx-routing-key");
channel.queueDeclare("myQueue", true, false, false, arguments);
```

#### 消息级 TTL

可以在发送消息时为每条消息设置 TTL，使用 `x-delay` 或者 `expiration` 参数来控制单个消息的存活时间。

```java
AMQP.BasicProperties props = new AMQP.BasicProperties.Builder()
    .expiration("60000")  // 设置消息的 TTL 为 60 秒
    .build();
channel.basicPublish("", "myQueue", props, message.getBytes());

```

### 3.5 死信队列

#### 消息变成死信的情况

1. 消息过期
2. 消息被手动拒收， requeue 为 false 且消息队列设置了死信交换机
3. 消息数量超过队列的最大长度

### 3.6 延迟队列

RabbitMQ 没有直接提供延迟队列，但可以通过结合 TTL 队列和死信交换机实现一个延迟队列。

### 3.7 日志与监控

### 3.8 消息可靠性分析与追踪

### 3.9 管理



## 4 RabbitMQ的应用问题
### 4.1 消息可靠性保障

#### 生产者弄丢消息

可以开启**发布确认**模式，当交换机没有收到消息或者无法把消息路由到队列时，会调用相应的回调方法（一般是记录异常日志，很少重发）。

配置文件：

```yml
rabbitmq:
    publisher-returns: true
    publisher-confirm-type: correlated #新版本 publisher-confirms: true 已过时
```

绑定回调方法：

```java
@Configuration
@Slf4j
public class RabbitMQConfig {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void enableConfirmCallback() {
        //confirm 监听，当消息成功发到交换机 ack = true，没有发送到交换机 ack = false
        //correlationData 可在发送时指定消息唯一 id
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if(!ack){
                //记录日志、发送邮件通知、落库定时任务扫描重发
            }
        });
        
        //当消息成功发送到交换机没有路由到队列触发此监听
        rabbitTemplate.setReturnsCallback(returned -> {
            //记录日志、发送邮件通知、落库定时任务扫描重发
        });
    }
}

```

消息体的业务端持久化：可以把消息记录存储在数据库中，方便未来回溯



#### 消息队列弄丢消息

消息队列和发送消息都要设置为持久化



#### 消费者弄丢消息

一般需要开启手动确认

```java
@RabbitListener(queues = "queue")
public void listen(String object, Message message, Channel channel) {
    long deliveryTag = message.getMessageProperties().getDeliveryTag();
    log.info("正在消费：{},消息内容:{}", deliveryTag, object);
    try {
        /**
         * 执行业务代码...
         * */
        channel.basicAck(deliveryTag, false);
    } catch (IOException e) {
        log.error("签收失败", e);
        try {
            channel.basicNack(deliveryTag, false, true);
        } catch (IOException exception) {
            log.error("拒签失败", exception);
        }
    }
}

```



### 4.2 消息重复消费

消费者接收消息时，由于网络抖动、超时重传、确认丢失等原因，可能会出现消息重复消费的情况，需要一定的措施保证消息的幂等性。

#### 解决方法

> 首先需要保证消费者在发送消息是赋予消息全局唯一标识（UUID、redis计数器、雪花算法）

1. 消费消息前，先**检查该消息是否已经消费过**，如果消费过就不执行，可以使用 Redis 的 SetNX 实现

```java
    Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent("orderNo+couponId");
    //先检查这条消息是不是已经消费过了
    if (!Boolean.TRUE.equals(flag)) {
        return;
    }
    //执行业务...
    //消费过的标识存储到 Redis，10 秒过期
    stringRedisTemplate.opsForValue().set("orderNo+couponId","1", Duration.ofSeconds(10L));

```

2. 允许重复消费，但保证**重复的消息不会对数据产生影响**。例如，利用数据库的主键唯一性约束，重复的数据无法插入成功；也可以利用数据库的乐观锁机制，每次更新数据时携带一个版本号，只有版本号和数据库相同才能更新成功。

## 5 RabbitMQ的集群搭建
