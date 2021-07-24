# 消息中间件

​		消息队列中间件`(Message Queue Middleware`，简称为`MQ）`是指利用高效可靠的消息传递机制进行与平台无关的数据交流，并基于数据通信来进行分布式系

统的集成。通过提供消息传递和消息排队模型，它可以在分布式环境下扩展进程间的通信。适用于需要可靠的数据传送的分布式环境。其能在不同平台之间通信，

它常被用来屏蔽各种平台及协议之间的特性，实现应用程序之间的协同，其优点在于能够在客户和服务器之间提供同步和异步的连接，并且在任何时刻都可以将消

息进行传送或者存储转发，这也是它比远程过程调用更进步的原因。



## 模式

点对点模式

​		点对点模式是基于队列的，消息生产者发送消息到队列,消息消费者从队列中接收消息,队列的存在使得消息的异步传输成为可能。

发布订阅模式

​		发布订阅模式定义了如何向一个内容节点发布和订阅消息，这个内容节点称为主题，主题可以认为是消息传递的中介，消息发布者将消息发布到某个主题，而

消息订阅者则从主题中订阅消息。主题使得消息的订阅者与消息的发布者互相保持独立，不需要进行接触即可保证消息的传递，发布/订阅模式在消息的一对多广

播时采用。



## 作用

解耦

​		在项目启动之初来预测将来会碰到什么需求是极其困难的。消息中间件在处理过程中间插入了一个隐含的、基于数据的接口层，两边的处理过程都要实现这一

接口，这允许你独立地扩展或修改两边的处理过程，只要确保它们遵守同样的接口约束即可。



冗余（存储)

​		有些情况下，处理数据的过程会失败。消息中间件可以把数据进行持久化直到它们已经被完全处理，通过这一方式规避了数据丢失风险。在把一个消息从消息

中间件中删除之前，需要你的处理系统明确地指出该消息已经被处理完成，从而确保你的数据被安全地保存直到你使用完毕。



扩展性

​		因为消息中间件解耦了应用的处理过程，所以提高消息入队和处理的效率是很容易的，只要另外增加处理过程即可，不需要改变代码，也不需要调节参数。



削峰

​		在访问量剧增的情况下，应用仍然需要继续发挥作用，但是这样的突发流量并不常见。如果以能处理这类峰值为标准而投入资源，无疑是巨大的浪费。使用消

息中间件能够使关键组件支撑突发访问压力，不会因为突发的超负荷请求而完全崩溃。



可恢复性

​		当系统一部分组件失效时，不会影响到整个系统。消息中间件降低了进程间的耦合度，所以即使一个处理消息的进程挂掉，加入消息中间件中的消息仍然可以

在系统恢复后进行处理。



顺序保证

​		在大多数使用场景下，数据处理的顺序很重要，大部分消息中间件支持一定程度上的顺序性。



缓冲

​		在任何重要的系统中，都会存在需要不同处理时间的元素。消息中间件通过一个缓冲层来帮助任务最高效率地执行，写入消息中间件的处理会尽可能快速。该

缓冲层有助于控制和优化数据流经过系统的速度。



异步通信

​		在很多时候应用不想也不需要立即处理消息。消息中间件提供了异步处理机制，允许应用把一些消息放入消息中间件中，但并不立即处理它，在之后需要的时

候再慢慢处理。



## 安装

[https://www.cnblogs.com/fengyumeng/p/11133924.html](https://www.cnblogs.com/fengyumeng/p/11133924.html)

命令

```shell
rabbitmq-server -detached // 启动
rabbitmqctl stop // 停止
rabbitmqctl status // 状态
```



实例

```java
/*
		<dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.9.0</version>
        </dependency>
*/

public class RabbitProducer
{
    private static final String EXCHANGE_NAME = "exchange_demo";
    private static final String ROUTING_KEY = "routingkey_demo";
    private static final String QUEUE_NAME = "queue_demo";
    private static final String IP_ADDRESS = "192.168.58.129";
    private static final int PORT = 5672;

    public static void main(String[] args) throws Exception
    {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(IP_ADDRESS);
        factory.setPort(PORT);
        factory.setUsername("xiaoshanshan");
        factory.setPassword("179980");

        Connection connection = factory.newConnection(); // 创建连接
        Channel channel = connection.createChannel(); // 创建信道
        channel.exchangeDeclare(EXCHANGE_NAME,"direct",true,false,null); // 创建一个 type=direct、持久化、非自动删除的交换器
        channel.queueDeclare(QUEUE_NAME,true,false,false,null); // 创建一个持久化、非排他的、非自动删除的队列
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,ROUTING_KEY); // 将交换器与队列通过路由键绑定

        String message = "hello world!";
        channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes()); // 发送一条持久化消息

        channel.close();
        connection.close();

    }
}

public class RabbitConsumer
{
    private static final String QUEUE_NAME = "queue_demo";
    private static final String IP_ADDRESS = "192.168.58.129";
    private static final int PORT = 5672;

    public static void main(String[] args) throws Exception
    {
        Address[] addresses = new Address[]{
                new Address(IP_ADDRESS,PORT)
        };

        ConnectionFactory factory = new ConnectionFactory();
        factory.setUsername("xiaoshanshan");
        factory.setPassword("179980");

        Connection connection = factory.newConnection(addresses); // 这里与生产者的连接方式不同
        Channel channel = connection.createChannel();
        channel.basicQos(64); // 设置客户端最多接收未被ack的消息的个数
        Consumer consumer = new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("recv message: " + new String(body));
                try
                {
                    TimeUnit.SECONDS.sleep(1);
                }
                catch (Exception e)
                {
                    e.printStackTrace();
                }
                channel.basicAck(envelope.getDeliveryTag(),false);
            }
        };

        channel.basicConsume(QUEUE_NAME,consumer);

        TimeUnit.SECONDS.sleep(5);
        channel.close();
        connection.close();
    }
}
```



# 入门

​		`RabbitMQ`整体上是一个生产者与消费者模型，主要负责接收、存储和转发消息。从计算机术语层面来说，`RabbitMQ`模型更像是一种交换机模型。

![](image/QQ截图20210724083817.png)

生产者

​		投递消息的一方。生产者创建消息，然后发布到`RabbitMQ`中。消息一般可以包含2个部分：消息体和标签。消息体也可以称之为`payload`，在实际应用中，消

息体一般是一个带有业务逻辑结构的数据。消息的用来表述这条消息。生产者把消息交由`RabbitMQ`，`RabbitMQ`之后会根据标签把消息发送给感兴趣的消费者。



消费者

​		接收消息的一方。消费者连接到`RabbitMQ`服务器，并订阅到队列上。当消费者消费一条消息时，只是消费消息的消息体。在消息路由的过程中，消息的标签

会丢弃，存入到队列中的消息只有消息体，消费者也只会消费到消息体，也就不知道消息的生产者是谁，当然消费者也不需要知道。



服务节点

​		对于`RabbitMQ`来说，一个`RabbitMQ Broker`可以简单地看作一个`RabbitMQ`服务节点，或者`RabbitMQ`服务实例。大多数情况下也可以将一个`RabbitMQ `

`Broker`看作一台`RabbitMQ`服务器。

![](image/QQ截图20210724084919.png)

​		首先生产者将业务方数据进行可能的包装，之后封装成消息，发送`(AMQP`协议里这个动作对应的命令为`Basic.Publish)`到`Broker`中。消费者订阅并接收消息

`(AMQP`协议里这个动作对应的命令为`Basic.consume`或者`Basic.Get)`，经过可能的解包处理得到原始的数据，之后再进行业务处理逻辑。



队列

​		`RabbitMQ`中消息都只能存储在队列中，`RabbitMQ`的生产者生产消息并最终投递到队列中,消费者可以从队列中获取消息并消费。多个消费者可以订阅同一个

队列，这时队列中的消息会被平均分摊`(`即轮询`)`给多个消费者进行处理，而不是每个消费者都收到所有的消息并处理。并且`RabbitMQ`不支持队列层面的广播消

费。

![](image/QQ截图20210724085930.png)



交换器

​		真实情况下，生产者将消息发送到交换器，由交换器将消息路由到一个或者多个队列中。如果路由不到，或许会返回给生产者，或许直接丢弃。交换器的类型

有四种：

​				1、`fanout`

​						它把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中。

​				2、`direct`

​						它会把消息路由那些`BindingKey`和`RoutingKey`完全匹配的队列中

​				3、`topic`

​						与`direct`类型的交换器相似，也是将消息路由到`BindingKey`和`RoutingKey`相匹配的队列，规则：

​								1)：`RoutingKey`为一个点号`"."`分隔的字符串`(`被点号`"."`分隔开的每一段独立的字符串称为一个单词`)`。

​								2)：`BindingKey`和`Routing`一样也是点号`"."`分隔的字符串。

​								3)：`BindingKey`中可以存在两种特殊字符串`"*"`和`"#"`，其中`"*"`用于匹配一个单词，`"#"`用于匹配多规格单词。

​				4、`headers`

​						此类型的交换器不依赖于路由键的匹配规则来路由消息，而是根据发送的消息的消息内容中的`headers`属性进行匹配。在绑定队列和交换器时制定

​				一组键值对，当发送消息到交换器时，`RabbitMQ`会获取到该消息的`headers(`也是一个键值对的形式`)`，对比其中的键值对是否完全匹配队列和交换器

​				绑定时指定的键值对，如果完全匹配则消息会路由到该队列，否则不会路由到该队列。

![](image/QQ截图20210724090640.png)

 



路由键

​		生产者将消息发给交换器的时候，一般会指定一个`RoutingKey`，用来指定这个消息的路由规则，而这个`RoutingKey`需要与交换器类型和绑定键联合使用才能

最终生效。在交换器类型和绑定键固定的情况下，生产者可以在发送消息给交换器时，通过指定`RoutingKey`来决定消息流向哪里。





绑定

​		`RabbitMQ`中通过绑定将交换器与队列关联起来，在绑定的时候一般会指定一个绑定键，这样`RabbitMQ`就知道如何正确地将消息路由到队列了。

![](image/QQ截图20210724091602.png)

​		生产者将消息发送给交换器时，需要一个`RoutingKey`，当`BindingKey`和`RoutingKey`相匹配时，消息会被路由到对应的队列中。在绑定多个队列到同一个交

换器的时候，这些绑定允许使用相同的`BindingKey`。`BindingKey`并不是在所有的情况下都生效，它依赖于交换器类型。



​		无论是生产者还是消费者，都需要和`RabbitMQ Broker`建立连接，这个连接就是一条`TCP`连接。一旦TCP 连接建立起来，客户端紧接着可以创建一个`AMQP`

信道，每个信道都会被指派一个唯一的`ID`。信道是建立在连接之上的虚拟连接，`RabbitMQ`处理的每条`AMOP`指令都是通过信道完成的。

