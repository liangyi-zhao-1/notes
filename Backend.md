# Data Transfer Object(DTO)
简单来说Model面向业务，我们是通过业务来定义Model的。而DTO是面向界面UI，是通过UI的需求来定义的。

通过DTO我们实现了表现层与Model之间的解耦，表现层不引用Model，如果开发过程中我们的模型改变了，而界面没变，我们就只需要改Model而不需要去改表现层中的东西。
# Dependency Injection


# Inversion of Control

Transfer the control of objects or portions of a program to a container or framework.

# Message Queue
A message queue is a form of asynchronous service-to-service communication used in serverless and microservices architectures. Messages are stored on the queue until they are processed and deleted. Each message is processed only once, by a single consumer. Message queues can be used to decouple heavyweight processing, to buffer or batch work, and to smooth spiky workloads.

消息队列是典型的：生产者、消费者模型。生产者不断向消息队列中生产消息，消费者不断的从队列中获取消息。因为消息的生产和消费都是异步的，而且只关心消息的发送和接收，没有业务逻辑的侵入，这样就实现了生产者和消费者的解耦。

结合前面所说的问题：

商品服务对商品增删改以后，无需去操作索引库或静态页面，只是发送一条消息，也不关心消息被谁接收。

搜索服务和静态页面服务接收消息，分别去处理索引库和静态页面。

如果以后有其它系统也依赖商品服务的数据，同样监听消息即可，商品服务无需任何代码修改。

现在实现MQ的有两种主流方式：AMQP、JMS。
两者间的区别和联系：

JMS是定义了统一的接口，来对消息操作进行统一；AMQP是通过规定协议来统一数据交互的格式

JMS限定了必须使用Java语言；AMQP只是协议，不规定实现方式，因此是跨语言的。

JMS规定了两种消息模型；而AMQP的消息模型更加丰富
ActiveMQ：基于JMS， Apache

RabbitMQ：基于AMQP协议，erlang语言开发，稳定性好

RocketMQ：基于JMS，阿里巴巴产品，目前交由Apache基金会

Kafka：分布式消息系统，高吞吐量

# RabbitMQ
## 基本消息模型
P：生产者，也就是要发送消息的程序

C：消费者：消息的接受者，会一直等待消息到来。

queue：消息队列，图中红色部分。类似一个邮箱，可以缓存消息；生产者向其中投递消息，消费者从其中取出消息。
## 消费者的消息确认机制
通过刚才的案例可以看出，消息一旦被消费者接收，队列中的消息就会被删除。

这就要通过消息确认机制（Acknowlege）来实现了。当消费者获取消息后，会向RabbitMQ发送回执ACK，告知消息已经被接收。不过这种回执ACK分两种情况：

自动ACK：消息一旦被接收，消费者自动发送ACK

手动ACK：消息接收后，不会发送ACK，需要手动调用

## work消息模型
让多个消费者绑定到一个队列，共同消费队列中的消息。队列中的消息一旦消费，就会消失，因此任务是不会被重复执行的。
我们可以修改设置，让消费者同一时间只接收一条消息，这样处理完成之前，就不会接收更多消息，就可以让处理快的人，接收更多消息 ：

## 订阅模型分类
而在订阅模型中，多了一个exchange角色，而且过程略有变化：

P：生产者，也就是要发送消息的程序，但是不再发送到队列中，而是发给X（交换机）

C：消费者，消息的接受者，会一直等待消息到来。

Queue：消息队列，接收消息、缓存消息。

Exchange：交换机，图中的X。一方面，接收生产者发送的消息。另一方面，知道如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于Exchange的类型。Exchange有以下3种类型：

Fanout：广播，将消息交给所有绑定到交换机的队列

Direct：定向，把消息交给符合指定routing key 的队列

Topic：通配符，把消息交给符合routing pattern（路由模式） 的队列
Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失！
 
### 订阅模型-Fanout
Fanout，也称为广播。
在广播模式下，消息发送流程是这样的：

1） 可以有多个消费者

2） 每个消费者有自己的queue（队列）

3） 每个队列都要绑定到Exchange（交换机）

4） 生产者发送的消息，只能发送到交换机，交换机来决定要发给哪个队列，生产者无法决定。

5） 交换机把消息发送给绑定过的所有队列

6） 队列的消费者都能拿到消息。实现一条消息被多个消费者消费

### 订阅模型-Direct
说明
在Fanout模式中，一条消息，会被所有订阅的队列都消费。但是，在某些场景下，我们希望不同的消息被不同的队列消费。这时就要用到Direct类型的Exchange。

在Direct模型下：

队列与交换机的绑定，不能是任意绑定了，而是要指定一个RoutingKey（路由key）

消息的发送方在 向 Exchange发送消息时，也必须指定消息的 RoutingKey。

Exchange不再把消息交给每一个绑定的队列，而是根据消息的Routing Key进行判断，只有队列的Routingkey与消息的 Routing key完全一致，才会接收到消息

### 订阅模型-Topic
说明
Topic类型的Exchange与Direct相比，都是可以根据RoutingKey把消息路由到不同的队列。只不过Topic类型Exchange可以让队列在绑定Routing key 的时候使用通配符！

 

Routingkey 一般都是有一个或多个单词组成，多个单词之间以”.”分割，例如： item.insert

通配符规则：

#：匹配一个或多个词

*：匹配不多不少恰好1个词

 

举例：

audit.#：能够匹配audit.irs.corporate 或者 audit.irs

audit.*：只能匹配audit.irs

## 持久化

如何避免消息丢失？

1） 消费者的ACK机制。可以防止消费者丢失消息。

2） 但是，如果在消费者消费之前，MQ就宕机了，消息就没了。

 

是可以将消息进行持久化呢？

要将消息持久化，前提是：队列、Exchange都持久化

解决消息丢失？

ack（消费者确认）

持久化

发送消息前，将消息持久化到数据库，并记录消息状态（可靠消息服务）

生产者确认（publisher confirm）
