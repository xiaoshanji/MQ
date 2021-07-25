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
        
        // factory.setUri("amqp://userName:password@ipAddress:portNumber/virtualHost"); // 上面连接参数的设置也可以使用本语句代替

        Connection connection = factory.newConnection(); // 创建连接
        Channel channel = connection.createChannel(); // 创建信道，多线程间 Channel 是不安全的
        // 判断连接或者信道是否打开，不推荐使用 isOpen 方法，通常使用捕获 ShutdownSignalException 来判断信道是否关闭，捕获IOException或SocketException来判断连接是否关闭
        
        channel.exchangeDeclare(EXCHANGE_NAME,"direct",true,false,null); // 创建一个 type=direct、持久化、非自动删除的交换器
        /*
DeclareOk exchangeDeclare(String exchange, String type, boolean durable, boolean autoDelete, boolean internal, Map<String, Object> arguments) throws IOException;
		exchange:交换器的名称
		type:交换器的类型
		durable：是否持久化，true表示持久化，持久化可以将交换器存盘，在服务器重启时不会丢失信息
		autoDelete：是否自动删除，自动删除的前提是至少有一个队列或者交换器与这个交换器绑定，之后所有与这个交换器绑定的队列或者交换器都与此解绑。
		internal：是否为内置的，如果为内置的，那么客户端无法直接发送消息到这个交换器中，只能通过交换器路由到交换器这种方式
		arguments：其他一些结构化参数

void exchangeDeclareNoWait(String var1, String var2, boolean var3, boolean var4, boolean var5, Map<String, Object> var6) throws IOException;
		此方法是上面的方法基础上增加了nowait参数，即不需要服务器返回，此时如果未创建成功，并且紧接着使用时会抛出异常，所以不推荐使用

DeclareOk exchangeDeclarePassive(String var1) throws IOException;
		检测交换器是否存在，存在则正常返回，不存在则抛出异常，同时信道会被关闭

DeleteOk exchangeDelete(String exchange, boolean ifUnuserd) throws IOException;
void exchangeDeleteNoWait(String exchange, boolean ifUnuserd) throws IOException;
DeleteOk exchangeDelete(String exchange) throws IOException;
		上面三个方法都用来删除交换器，exhange：交换器的名称，ifUnuserd：是否在交换器没有被使用的情况下删除
        */
        
        
        channel.queueDeclare(QUEUE_NAME,true,false,false,null); // 根据指定的名称创建一个持久化、非排他的、非自动删除的队列，连接断开时不会删除
        /*
        DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments) throws IOException;
        	queue：队列名称
        	durable：是否持久化
        	exclusive：是否排他，如果声明为排他，那么该队列仅对首次声明它的连接可见，连接断开时自动删除，排他队列是基于连接可见的，同一个连接的不同信道是可以同时访问同一连接创建的排他队列，并且当一个连接声明了一个排他队列，那么其他连接是不允许建立同名的排他队列，即使是持久化的，一旦连接关闭或者客户端退出，该排他队列也会自动删除，适用于一个客户端同时发送或读取消息的应用场景。
        	autoDelete：是否自动删除，前提是至少有一个消费者连接到这个队列，之后所有与这个队列连接的的消费者都断开时，才会自动删除。
        	arguments：其他一些参数。
        
        如果消费者在同一个信道上想更改声明队列，那么必须先取消订阅，然后将信道置为传输模式，之后才能声明队列
        
        void queueDeclareNoWait(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments) throws IOException;
        	此方法同样在声明队列时无需等待服务端返回，所以跟交换器一样也有可能发生异常情况
        
        DeclareOk queueDeclarePassive(String queue) throws IOException;
        	检测队列是否存在，有则正常返回，没有则抛出异常
        
        DeleteOk queueDelete(String queue) throws IOException;
    	DeleteOk queueDelete(String queue, boolean ifUnused, boolean ifEmpty) throws IOException;
        void queueDeleteNoWait(String queue, boolean ifUnused, boolean ifEmpty) throws IOException;
        	同样上面的三个方法用于删除队列，queue：队列名称，ifUnused：与交换器的作用相同，ifEmpty：设置为true，则表示在队列为空时删除队列
        
        PurgeOk queuePurge(String queue) throws IOException;
        	清除队列的内容，不删除队列本身
        	
        */
        // String QUEUE_NAME = channel.queueDeclare().getQueue(); // 创建了一个非持久化，排他的，自动删除的队列，此队列也称之为匿名队列，名称自动生成，并且该队列在连接断开时会自动删除
        
        /*
        	如果声明一个已经存在的交换器或队列，只要声明参数完全匹配现存的交换器或者队列，RabbitMQ将直接成功返回，如果参数不匹配则会抛出异常
        */
        
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,ROUTING_KEY); // 将交换器与队列通过路由键绑定
        /*
        BindOk queueBind(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;
        	queue：队列名称
        	exchange：交换器名称
        	routingKey：绑定队列和交换器的路由键
        	arguments：定义绑定的其他参数
        同样该方法也有一个不需要等待的方法
        	
        UnbindOk queueUnbind(String queue, String exchange, String routingKey, Map<String, Object> arguments) throws IOException;
        	将队列与交换器解绑
        
        BindOk exchangeBind(String destination, String source, String routingKey, Map<String, Object> arguments) throws IOException;
        将交换器与交换器绑定，绑定之后消息从 source 交换器转发到 destination 交换器，此时可以将 destination 交换器看作一个队列
        	
        
        
        
        */

        String message = "hello world!";
        channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes()); // 发送一条持久化消息
        /*
        void basicPublish(String exchange, String routingKey, BasicProperties props, byte[] body) throws IOException;
        void basicPublish(String exchange, String routingKey, boolean mandatory, BasicProperties props, byte[] body) throws IOException;
        void basicPublish(String exchange, String routingKey, boolean mandatory, boolean immediate, BasicProperties props, byte[] body)
            throws IOException;
        exchange：交换器名称，指明消息需要发送到哪个交换器中，如果为空字符串，则会发送到RabbitMQ默认的交换器中
        routingKey：路由键，交换器根据路由键将消息存储到相应的队列中
        props：消息的基本属性集，包含14个属性成员：contentType、contentEncoding、headers、deliveryMode、priority、correlationId、replyTo、expiration、messageId、timestamp、type、userId、appId、clusterId;
        body：消息体，真正需要发送的消息
        
        
        */

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
                channel.basicAck(envelope.getDeliveryTag(),false); // 确认消息被成功接收
                /*
                void basicReject(long deliveryTag, boolean requeue) throws IOException;
                拒绝单条消息
                	deliveryTag：可以看作消息的编号
                	requeue：如果为true，则该消息会重新存入队列，false则会从队列中移除该消息
                
                void basicNack(long deliveryTag, boolean multiple, boolean requeue) throws IOException;
                批量拒绝消息
                	multiple：为false，则表示拒绝编号为deliveryTag这一条消息，如果为true则表示拒绝deliveryTag编号之前所有未被当前消费者确认的消息
                
                
                RecoverOk basicRecover() throws IOException;
                RecoverOk basicRecover(boolean requeue) throws IOException;
                请求RabbitMQ重新发送还未被确认的消息
                	requeue：为true，则未被确认的消息会重新加入到队列中，消息可能会被分配到与之前不同的消费者；如果为false，消息会被分配到与之前相同的				消费者。
                */
            }
        };

        channel.basicConsume(QUEUE_NAME,consumer);
        /*
        String basicConsume(String queue, Consumer callback) throws IOException;
        String basicConsume(String queue, boolean autoAck, Consumer callback) throws IOException;
        String basicConsume(String queue, boolean autoAck, Map<String, Object> arguments, Consumer callback) throws IOException;
        String basicConsume(String queue, boolean autoAck, String consumerTag, Consumer callback) throws IOException;
        String basicConsume(String queue, boolean autoAck, String consumerTag, boolean noLocal, boolean exclusive, Map<String, Object> arguments, Consumer callback) throws IOException;
        上面的方法采用推模式消费消息，此时RabbitMQ会不断的推送消息给消费者，个数受到channel.basicQos()方法设置的限制
        	queue：队列名称
        	autoAck：是否自动确认，建议设置为false，此时需要调用channel.basicAck方法来确认消息被成功接收，RabbitMQ在收到确认消息之后才会从内存或磁盘		中移除消息，此时队列中的消息被分为两部分，一部分为等待投递的消息，一部分为投递了还没收到确认信号的消息。如果一直没有等到确认信号，并且消费消息的消费		者已经断开连接，那么该消息会重新进入队列，等待下一个消费者。如果为true，那么推送消息后RabbitMQ将自动为该消息打上确认标记。
        	consumerTag：消费者标签，用来区分多个消费者
        	noLocal：设置为true则表示不能将同一个连接中生产者发送的消息发送给这个连接中的消费者
        	exclusive：是否为排他
        	arguments：设置消费者的其他参数
        	callback：设置消费者的回调函数，用来处理RabbitMQ推送过来的消息。
        	
        采用拉模式，可以单条地获取消息
        GetResponse basicGet(String queue, boolean autoAck) throws IOException;
        
        不能将basicGet放在循环体里来替代basicConsume，会严重影响RabbitMQ的性能
        */

        TimeUnit.SECONDS.sleep(5);
        
        channel.close();
        connection.close();
        /*
        void close() throws IOException;
        void close(int closeCode, String closeMessage) throws IOException;
        显示通知当前对象执行关闭操作
        
        */
        
        /*
        
        Connection和Channel的生命周期：
        	Open：开启状态，标识当前对象可用
        	Closing:正在关闭状态。当前对象被显式地通知调用关闭方法，这样就产生了一个关闭请求让其内部对象进行相应的操作，并等待这些关闭操作的完成。			    	Closed:已经关闭状态。当前对象已经接收到所有的内部对象已完成关闭动作的通知，并且其也关闭了自身。

        可以在connection中注册关闭监听事件addShutdownListener(ShutdownListener listener)。当Connection和Channel转为Closed状态时会调用监听事件，		当将该事件注册到一个已经处于Closed的两个对象中，则会立刻调用
        
        
        ShutdownSignalException getCloseReason()
        获取对象关闭的原因，ShutdownSignalException对象的isHardError()方法可用来判断是Connection还是Channel的错误
        
        */
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



