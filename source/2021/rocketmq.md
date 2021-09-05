# 大厂如何使用 RocketMQ

## RocketMQ 集群架构图

![rocketmq_architecture_1](https://raw.githubusercontent.com/telzhou618/images/main/img/rocketmq_architecture_1.png)

## RocketMQ 四大核心集群

- **NameServer** 集群：作为注册中心，维护着所有的broker和topic的映射关系，以及路由功能，多台部署，之间不互相通信。
- **Broker集群** ：负责消息存储，消息转发。
- **Producer集群** ：生产消息，一般是自己的Application集群。
- **Consumer集群**：消费消息，一般是自己的Application集群。

## RocketMQ 其他核心组件

- **Consumer Group**：消费者组，集群模式相同Group的每个Consumer平均分摊消息，广播模式相同Group的每个Consumer都接收全量消息。
- **Topic**：一类消息的集合，一个消息只能属于一个Topic。
- **Message**：消息的载体。
- **Message Queue**：消息分区，每个Topic下可以有多个分区。
- **Tag**：给消息设置一个标签，消费者可以过滤出想要的消息。

## RocketMQ 消息发送方式

### 单项发送： 

只负责发出去，不管是否成功，没有返回值，一般用在可靠性不高的场景，如记录日志。

```java
producer.sendOneway(msg);
```

### 同步发送

需要同步等待消费发送的结果，有返回值，用在可靠性要求高，性能不高的场景。

```java
// Send message to one of brokers
SendResult sendResult = producer.send(msg);
// Check whether the message has been delivered by the callback of sendResult
System.out.printf("%s%n", sendResult);
```

### 异步发送

无需等待返回值，需要实现回调方法，消息发送完成会自动执行回调方法，用在可靠性和性能要求较高的场景。

```java
producer.send(msg, new SendCallback() {
  @Override
  public void onSuccess(SendResult sendResult) {
    System.out.printf("%-10d OK %s %n", index,
                      sendResult.getMsgId());
  }
  @Override
  public void onException(Throwable e) {
    System.out.printf("%-10d Exception %s %n", index, e);
    e.printStackTrace();
  }
});
```

## RocketMQ 消息类型

### 顺序消息

- 局部有序：需要生产者和消费者同时保证，只能把需要保证有序的消息发送到同一个MessageQuenue，在消费者端同一个MessageQuenue的消息只能由一个消费者消费。

  - 如：电商场景的下单业务，对于同一个丁当它从下单、付款、发货是有序的，但不同的订单之间又是无序的，可用这种方案。
  - 生产者端代码：实现 select 方法，把相同订单的消息发在同一个 quenue。

  ```java
  SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
     @Override
     public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
       Long id = (Long) arg;  //根据订单id选择发送queue
       long index = id % mqs.size();
       return mqs.get((int) index);
     }
   }, orderList.get(i).getOrderId());//订单id
  ```

  - 消费者端代码： 重要的是注册一个MessageListenerOrderly类型的消费者

  ```java
  consumer.registerMessageListener(new MessageListenerOrderly() {
    @Override
    public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
      context.setAutoCommit(true);
      for (MessageExt msg : msgs) {
        // 可以看到每个queue有唯一的consume线程来消费, 订单对每个queue(分区)有序
        System.out.println("consumeThread=" + Thread.currentThread().getName() + "queueId=" + msg.getQueueId() + ", content:" + new String(msg.getBody()));
      }
      return ConsumeOrderlyStatus.SUCCESS;
    }
  });
  ```

- 全局有序：一个Topic只能有一个MessageQuenue以及消费只能有一个消费者，一般不这么用。

### 事务消息

是指应用本地事务和消息可以定义在一个全局事务中，要么同时成功，要么同时失败，可应用在消息和数据库数据严格一致的场景。

样例：[https://github.com/apache/rocketmq/blob/master/docs/cn/RocketMQ_Example.md#61-发送事务消息样例](https://github.com/apache/rocketmq/blob/master/docs/cn/RocketMQ_Example.md#61-发送事务消息样例)

### 延迟消息

消息发送给brocker后不会立即消息，一段时间后投递给真正的 Topic， 比如：电商场景一个订单创建1小时候没付款，自动释放库存。

```java
Message message = new Message("TestTopic", ("Hello scheduled message " + i).getBytes());
// 设置延时等级3,这个消息将在10s之后发送(现在只支持固定的几个时间,详看delayTimeLevel)
message.setDelayTimeLevel(3);
// 发送消息
producer.send(message);
```

- 使用限制

```java
private String messageDelayLevel = "1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h";
```

### 可靠消息

- 如何保证生产者发消息成功？同步发消息、同步方式同步从节点、同步方式刷盘。

- 如何保证消费者消费成功？不要异步消费，消费成功返回ACK，如果消费失败，MQ会重试。

- MQ 集群宕机怎么办？把消息咱存在Redis，等MQ恢复后再异步再发给MQ。

### 消息过滤

发消息时设置 TAG，消费端可以用TAG过滤只选择自己想要的消息，不想要的不会发给消费者，可以减少网络传输，但增加复杂性。

### 回溯消息

消费过的消息不会立即删除，如果MQ宕机重启后，消费者可以选择重新消费某一时间前的历史消息重新消费，称为回溯消息。

### 流量控制

如果 broker 的处理能达到瓶颈，可设置拒绝消息发送，消费者端可通过降低拉取消息的评率流控。

### 死信队列

对于消费失败的消息，重试次数达到最大值后会迁移到一个特殊的队列，这类消息称为死信，可以在MQ控制台手动重发来进行消费。

## RocketMQ 核心原理

### 事务消息原理

### Brocker 选主原理

### Brocker 存储消息原理

![rocketmq_design_1](https://raw.githubusercontent.com/telzhou618/images/main/img/rocketmq_design_1.png)

- CommitLog：具体存储消息的载体，提前申请连续存储空间，顺序写入速度快。
- ConsumeQueue：消费队列，记录Topic和消息存储关系，已经记录消息的当前消费位置（偏移量）。
- IndexFile：索引文件，可通过KEY或时间区间快速查询消息。

## RocketMQ 对比其他MQ

- RabbitMQ 安全性高，可靠性高，但吞吐量低。
- Kafka 性能高，但功能单一，且可能会丢消息。
- RocketMQ 性能高，功能全，几乎全场景都能使用。
