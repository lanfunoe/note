# Rabbitmq

## Hello World

| 标志 | 中文名     | 英文名   | 描述                                             |
| ---- | ---------- | -------- | ------------------------------------------------ |
| P    | 生产者     | Producer | 消息的发送者，可以将消息发送到交换机             |
| C    | 消费者     | Consumer | 消息的接收者，从队列中获取消息进行消费           |
| X    | 交换机     | Exchange | 接收生产者发送的消息，并根据路由键发送给指定队列 |
| Q    | 队列       | Queue    | 存储从交换机发来的消息                           |
| type | 交换机类型 | type     | direct表示直接根据路由键（orange/black）发送消息 |

RabbitMQ is a message broker

功能类比于邮局，但是accepts, stores, and forwards binary blobs of data ‒ *messages*.

生产者：发送消息

队列是一种邮箱，虽然消息经过消息队列和应用程序，但是仅存储在队列中，队列仅受主机的内存和磁盘限制的约束，本质上是一个大型消息缓冲区。

消费者：等待，接受消息

不必在同一主机，但是一个应用程序可以同时是消费者和生产者



> Note we can use a try-with-resources statement because both Connection and Channel implement java.io.Closeable. This way we don't need to close them explicitly in our code.

it will only be created if it doesn't exist already. The message content is a byte array, so you can encode whatever you like there.

## Work Queues

The main idea behind Work Queues (aka: *Task Queues*) is to avoid doing a resource-intensive task immediately and having to wait for it to complete

### preparation

### Message acknowledgment

An acknowledgement is sent back by the consumer to tell RabbitMQ that a particular message has been received, processed and that RabbitMQ is free to delete it.

一旦没有，就会重新分配给其他消费者

默认30min强制发送。

[Manual message acknowledgments](https://www.rabbitmq.com/confirms.html) are turned on by default.

通过autoAck=true关闭

### 消息的持续性

Two things are required to make sure that messages aren't lost: we need to mark both the queue and messages as durable.

```java
boolean durable = true;
channel.queueDeclare("hello", durable, false, false, null);
```

RabbitMQ doesn't allow you to redefine an existing queue with different parameters and will return an error to any program that tries to do that. 

所以改动是错误的

可持续性：queueDeclare change needs to be applied to both the producer and consumer code.

we need to mark our messages as persistent - by setting MessageProperties (which implements BasicProperties) to the value PERSISTENT_TEXT_PLAIN.

```java
channel.basicPublish("", "task_queue",
            MessageProperties.PERSISTENT_TEXT_PLAIN,
            message.getBytes());
```

Although it tells RabbitMQ to save the message to disk, there is still a short time window when RabbitMQ has accepted a message and hasn't saved it yet. Also, RabbitMQ doesn't do fsync(2) for every message -- it may be just saved to cache and not really written to the disk.  If you need a stronger guarantee then you can use [publisher confirms](https://www.rabbitmq.com/confirms.html).

### Fair dispatch

In order to defeat that we can use the basicQos method with the prefetchCount = 1 setting. This tells RabbitMQ not to give more than one message to a worker at a time. Or, in other words, don't dispatch a new message to a worker until it has processed and acknowledged the previous one.

## Publish/Subscribe

The core idea in the messaging model in RabbitMQ is that the producer never sends any messages directly to a queue. Actually, quite often the producer doesn't even know if a message will be delivered to any queue at all.

### Exchanges

作用：

1.it receives messages from producers 

2.it pushes them to queues.

决定怎么处理生产者的消息，取决于exchange类型

There are a few exchange types available: direct, topic, headers and fanout.

Let's create an exchange of this type, and call it logs:

```java
channel.exchangeDeclare("logs", "fanout");
```

it just broadcasts all the messages it receives to all the queues it knows. 

In previous parts of the tutorial we knew nothing about exchanges, but still were able to send messages to queues. That was possible because we were using a default exchange, which we identify by the empty string ("").

```java
channel.basicPublish("", "hello", null, message.getBytes());
```

The first parameter is the name of the exchange. The empty string denotes the default or *nameless* exchange: messages are routed to the queue with the name specified by routingKey, if it exists.

### Temporary queues

使用场景：We want to hear about all log messages, not just a subset of them. We're also interested only in currently flowing messages not in the old ones. 

To solve that we need two things.

Firstly, whenever we connect to Rabbit we need a fresh, empty queue. To do this we could create a queue with a random name, or, even better - let the server choose a random queue name for us.

Secondly, once we disconnect the consumer the queue should be automatically deleted.

In the Java client, when we supply no parameters to queueDeclare() we create a non-durable, exclusive, autodelete queue with a generated name:

```java
String queueName = channel.queueDeclare().getQueue();
```

At that point queueName contains a random queue name. For example it may look like amq.gen-JzTY20BRgKO-HjmUJj0wLg

### Bindings

That relationship between exchange and a queue is called a *binding*.

![img](resource/bindings.png)

```java
channel.queueBind(queueName, "logs", "");
```

From now on the logs exchange will append messages to our queue.

## Routing

### Bindings

The meaning of a binding key depends on the exchange type. The fanout exchanges, which we used previously, simply ignored its value.

### Direct exchange

direct exchange is simple - a message goes to the queues whose binding key exactly matches the routing key of the message.

![img](resource\direct-exchange.png)

### Multiple bindings

### Emitting logs

As always, we need to create an exchange first:

```java
channel.exchangeDeclare(EXCHANGE_NAME, "direct");
```

And we're ready to send a message:

```java
channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes());
```

### Subscribing

```java
String queueName = channel.queueDeclare().getQueue();

for(String severity : argv){
  channel.queueBind(queueName, EXCHANGE_NAME, severity);
}
```

![img](resource\python-four.png)

## Topics

 routing_key - it must be a list of words, delimited by dots.

 limit of 255 bytes.



- \* (star) can substitute for exactly one word.
- \# (hash) can substitute for zero or more words.

![img](resource\python-five.png)

## Remote procedure call (RPC)

The problems arise when a programmer is not aware whether a function call is local or if it's a slow RPC.

advice:

- Make sure it's obvious which function call is local and which is remote.
- Document your system. Make the dependencies between components clear.
- Handle error cases. How should the client react when the RPC server is down for a long time?

When in doubt avoid RPC. If you can, you should use an asynchronous pipeline - instead of RPC-like blocking, results are asynchronously pushed to a next computation stage.

### Callback queue

在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，如果发现该消息不可路由，那么消息会被直接丢弃，此时生产者是不知道消息被丢弃这个事件的。

 Most of the properties are rarely used, with the exception of the following:

- deliveryMode: Marks a message as persistent (with a value of 2) or transient (any other value). You may remember this property from [the second tutorial](https://www.rabbitmq.com/tutorials/tutorial-two-java.html).
- contentType: Used to describe the mime-type of the encoding. For example for the often used JSON encoding it is a good practice to set this property to: application/json.
- replyTo: Commonly used to name a callback queue.
- correlationId: Useful to correlate RPC responses with requests.

### Correlation Id

In the method presented above we suggest creating a callback queue for every RPC request. That's pretty inefficient, but fortunately there is a better way - let's create a single callback queue per client.

That raises a new issue, having received a response in that queue it's not clear to which request the response belongs.

That's when the correlationId property is used

based on that we'll be able to match a response with a request.

### Summary

![img](resource\python-six.png)

Our RPC will work like this:

- For an RPC request, the Client sends a message with two properties: replyTo, which is set to an anonymous exclusive queue created just for the request, and correlationId, which is set to a unique value for every request.
- The request is sent to an rpc_queue queue.
- The RPC worker (aka: server) is waiting for requests on that queue. When a request appears, it does the job and sends a message with the result back to the Client, using the queue from the replyTo field.
- The client waits for data on the reply queue. When a message appears, it checks the correlationId property. If it matches the value from the request it returns the response to the application.

## Publisher Confirms

[官方 RabbitMQ 教程 - 7 Publisher Confirms_一线大码的博客-CSDN博客](https://blog.csdn.net/wb1046329430/article/details/115296159)

Confirms should be enabled just once, not for every message published.

```java
Channel channel = connection.createChannel();
channel.confirmSelect();
```

最后一部分，待续

