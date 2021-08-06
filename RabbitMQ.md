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



# 进阶

```java
void basicPublish(String exchange, String routingKey, boolean mandatory, boolean immediate, BasicProperties props, byte[] body) throws IOException;
```

​		`mandatory`和`immediate`都有当消息传递过程中不可达目的地时将消息返回给生产者的功能。`RabbitMQ`提供的备份交换器可以将未能被交换器路由的消息

`(`没有绑定队列或者没有匹配的绑定`)`存储起来，而不用返回给客户端。

​		当`mandatory`设为`true`时，交换器无法根据自身的类型和路由键找到一个符合条件的队列，那么`RabbitMQ`会将消息返回给生产者。当`mandatory`参数设置为

`false`时，出现上述情形，则消息直接被丢弃。可以通过调用`channel.addReturnListener`来添加`ReturnListener`监听器实现监听消息是否被路由到合适的队列。

​		当`immediate`参数设为`true`时，如果交换器在将消息路由到队列时发现队列上并不存在任何消费者，那么这条消息将不会存入队列中。当与路由键匹配的所

有队列都没有消费者时，该消息会返回至生产者。但此参数在`RabbitMQ 3.0`开始就不在被支持。即当调用`basicPublish`方法将`immediate`置为`true`时，程序会

出现异常。



​		使用`mandatory`参数加监听事件处理未被正确路由的消息过于繁琐，`RabbitMQ`提供了备份交换器，可以将未被路由的消息存储在其中，有需要的时候再去处

理这些消息。

​		备份交换器可以在声明交换器的时候添加`alternate-exchange`参数来实现，也可以通过策略实现，两者同时使用，前者会覆盖后者。消息被重新发送到备份

交换器时的路由键和从生产者发出的路由键是一样的。即当消息到达备份交换器时，备份交换器拿到的路由键同样是生产者发送的路由键，然后备份交换器在根据

自己的规则和这个路由键来处理消息。

```java
		Map<String,Object> arguments = new HashMap<>();
        arguments.put("alternate-exchange","Ae");

        channel.exchangeDeclare("normalExchange",BuiltinExchangeType.DIRECT,true,false,arguments);
        channel.exchangeDeclare("Ae",BuiltinExchangeType.FANOUT,true,false,null);

        channel.queueDeclare("normalQueue",true,false,false,null);
        channel.queueBind("normalQueue","normalExchange","normalKey");

        channel.queueDeclare("unroutedQueue",true,false,false,null);
        channel.queueBind("unroutedQueue","Ae","");
```

​		1、如果设置的备份交换器不存在，客户端和`RabbitMQ`服务端都不会有异常出现，此时消息会丢失。

​		2、如果备份交换器没有绑定任何队列，客户端和`RabbitMQ`服务端都不会有异常出现，此时消息会丢失。

​		3、如果备份交换器没有任何匹配的队列，客户端和`RabbitMQ`服务端都不会有异常出现，此时消息会丢失。

​		4、如果备份交换器和`mandatory`参数一起使用，那么`mandatory`参数无效。



## 过期时间

​		`RabbitMQ`可以对消息和队列设置过期时间。

### 消息过期时间

​		两种设置方法：两种方法同时设置，以两者之间较小的数值为准。

​				1、通过队列属性设置，队列中所有消息都有相同的过期时间。在`channel.queueDeclare()`方法中加入`x-message-ttl`参数实现，单位为毫秒。如果不设

​		置过期时间，则代表消息不会过期。设置为`0`，表示除非此时可以直接将消息投递给消费者，否则该消息会被立即丢弃。一旦消息过期，此时已过期的消息

​		会在队列头部，马上就会从队列中抹去。

​				2、对消息本身进行单独设置。通过`channel.basicPublish()`方法中加入`expiration`属性，单位为毫秒。消息过期，并不会马上从队列中抹去，每条消

​		息是否过期是在即将投递到消费者之前判定的。

```java
		AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
        builder.deliveryMode(2); // 持久化消息
        builder.expiration("60000"); // 设置过期时间
        AMQP.BasicProperties properties = builder.build();
        channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY, true,properties,message.getBytes()); // 发送一条持久化消息
```



### 队列过期时间

​		通过`channel. queueDeclare`方法中的`x-expires`参数可以控制队列被自动删除前处于未使用状态的时间，约束条件可以参照上面`x-message-ttl`参数，但是

该参数不能设置为`0`。未使用的意思是队列上没有任何的消费者，队列也没有被重新声明，并且在过期时间段内也未调用过`basicGet`方法。

​		`RabbitMQ`会确保在过期时间到达后将队列删除，但是不保障删除的动作有多及时。在`RabbitMQ`重启后，持久化的队列的过期时间会被重新计算。



## 死信队列

​		也可以称之为死信交换器，当一个消息在一个队列中变成死信`(`超过过期时间的消息`)`之后，它能被重新被发送到另一个交换器中，即死信交换器。绑定死

信交换器的队列被称为死信队列。

​		几种情况：

​				1、消息被拒绝，并且设置`requeue`参数为`false`。

​				2、消息过期。

​				3、队列达到最大长度。

​		死信交换器也是一个正常的交换器，和一般的交换器没有区别，它能在任何的队列上被指定，实际上就是设置某个队列的属性`(x-dead-letter-exchange)`。

当这个队列中存在死信时，`RabbitMQ`就会自动地将这个消息重新发布到设置的死信交换器上去，进而被路由到另一个队列，即死信队列。可以监听这个队列中的

消息以进行相应的处理，这个特性与将消息的过期时间设置为`0`配合使用可以弥补`immediate`参数的功能。并且可以通过`x-dead-letter-routing-key`为死信交换

器指定路由键，如果没有指定则使用原队列的路由键。

![](image/QQ截图20210726180822.png)





## 延迟队列

​		延迟队列存储的对象是对应的延迟消息，所谓延迟消息是指当消息被发送以后，并不想让消费者立刻拿到消息，而是等待特定时间后，消费者才能拿到这个消

息进行消费。

​		`RabbitMQ`本身并没有直接支持延迟队列的功能。但是可以通过死信队列和过期时间模拟出来，此时死信队列即被视为延迟队列。而消费者也直接订阅到死信

队列上，此时原本队列上的消息会因为过期，而转入相应的死信队列，消费者再从死信队列中消费消息，从而实现延迟队列。

![](image/QQ截图20210726180734.png)



## 优先级队列

​		具有高优先级的队列具有高的优先权，优先级高的消息具备优先被消费的特权。可以通过设置队列的`x-max-priority`参数来实现。

```java
		// 配置队列的最大优先级
		Map<String,Object> arguments = new HashMap<>();
        arguments.put("x-max-priority",10);
        channel.queueDeclare("queue.priority",true,false,false,arguments);

		// 设置消息的优先级
		AMQP.BasicProperties.Builder builder = new AMQP.BasicProperties.Builder();
        builder.priority(5);
        AMQP.BasicProperties properties = builder.build();
        channel.basicPublish("exchange","routingKey",properties,"message".getBytes());
```

​		如果在消费者的消费速度大于生产者的速度并且服务节点中没有消息堆积的情况下，对发送的消息设置优先级没有实际意义。



## RPC实现

​		`RPC`：远程过程调用，通过网络从远程计算机上请求服务。

​		通过`RabbitMQ`进行`RPC`，首先客户端发送请求消息，服务端回复响应的信息，为了能够接收响应的消息，需要在请求消息中发送一个回调队列。并且为了区

别当前消息对应哪一个请求，每个请求应该带有唯一的请求`ID`。之后回到队列接收到回复的消息时，可以根据这个`ID`匹配到相应的请求。如果收到一条未知`ID`

的回复消息，可以简单地将其丢弃。

![](image/QQ截图20210726184826.png)

​		处理流程：

​				1、当客户端启动时，创建一个匿名的回调队列`(`名称由`RabbitMQ`自动创建`)`。

​				2、客户端为`RPC`请求设置2个属性：`replyTo`用来告知`RPC`服务端回复请求时的目的队列，即回调队列；`correlationId`用来标记一个请求。

​				3、请求被发送到`rpc_queue`队列中。

​				4、`RPC`服务端监听`rpc_queue`队列中的请求，当请求到来时，服务端会处理并且把带有结果的消息发送给客户端。接收的队列就是`replyTo`设定的回调

​		队列。



## 持久化

​		持久化可以提高`RabbitMQ`的可靠性，以防在异常情况`(`重启、关闭、宕机等`)`下的数据丢失。`RabbitMQ`的持久化分为三个部分：交换器的持久化、队列的

持久化和消息的持久化。

​		交换器的持久化是通过在声明交换器是将`durable`参数置为`true`实现的。如果交换器不设置持久化，那么在`RabbitMQ`服务重启之后，相关的交换器元数据会

丢失，不过消息不会丢失，只是不能将消息发送到这个交换器中了。

​		队列的持久化是通过在声明队列时将`durable`参数置为`true`实现的。如果队列不设置持久化，那么在`RabbitMQ`服务重启之后，相关队列的元数据会丢失，此

时数据也会丢失。

​		队列的持久化能保证其本身的元数据不会因异常情况而丢失，但是并不能保证内部所存储的消息不会丢失。要确保消息不会丢失，需要将其设置为持久化。通

过将消息的投递模式`(BasicProperties中的deliveryMode`属性`)`设置为`2`即可实现消息的持久化。

​		设置了队列和消息的持久化，当`RabbitMQ`服务重启之后，消息依旧存在。单单只设置队列持久化，重启之后消息会丢失;单单只设置消息的持久化，重启之后

队列消失，继而消息也丢失。如果将所有的消息都设置为持久化，将会严重影响`RabbitMQ`的性能。

​		即使将交换器、队列、消息都设置为持久化也不能保证消息不丢失：如果消费者将`autoAck`参数置为`true`，在收到消息后还没处理就宕机了，此时数据也丢

失了。其次，在持久化的消息存入`RabbitMQ`后，还需要一段很短的时间才能存入磁盘，`RabbitMQ`并不会为每条消息进行同步存盘，如果这段时间服务节点发生宕

机，重启等情况，消息没来得及存盘，那么数据同样会丢失。

​		要解决上述问题，可以通过镜像队列机制和发送端引入事务机制或发送方确认机制。



## 生产者确认

​		默认情况下发送消息的操作是不会返回任何信息给生产者的，即生产者是不知道消息有没有正确地到达服务器。如果在消息到达服务器之前已经丢失，持久化

操作也解决不了这个问题，`RabbitMQ`针对这个问题，提供了两种解决方式：

​				1、通过事务机制实现。

​				2、通过发送方确认机制实现。

### 事务机制

​		`RabbitMQ`客户端中与事务机制相关的方法有三个：

​				`channel.txSelect`：将信道设置为事务模式。

​				`channel.txCommit`：提交事务。

​				`channel.txRollback`：事务回滚。

```java
		try
        {
            channel.txSelect();
            channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes()); // 发送一条持久化消息
            channel.txCommit();
        }
        catch (Exception e)
        {
            e.printStackTrace();
            channel.txRollback();
        }
```

![](image/QQ截图20210727083336.png)



​		发送多条消息：

```java
		channel.txSelect();
        for(int i = 0 ; i < length ; i++)
        {
            try
            {

                channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes()); // 发送一条持久化消息
                channel.txCommit();
            }
            catch (Exception e)
            {
                e.printStackTrace();
                channel.txRollback();
            }
        }
```

​		事务机制在一条消息发送之后会使发送端阻塞，以等待`RabbitMQ`的回应，之后才能继续发送下一条消息。

​		事务确实能够解决消息发送方和`RabbitMQ`之间消息确认的问题，只有消息成功被`RabbitMQ`接收，事务才能提交成功，否则便可在捕获异常之后进行事务回

滚，与此同时可以进行消息重发。但是使用事务机制会吸干`RabbitMQ`的性能。



### 发送方确认

​		生产者将信道设置成确认模式，所有在该信道上面发布的消息都会被指派一个唯一的`ID(`从`1`开始`)`，一旦消息被投递到所有匹配的队列之后，`RabbitMQ`就

会发送一个确认给生产者`(`包含消息的唯一`ID)`，这就使得生产者知晓消息已经正确到达了目的地了。如果消息和队列是可持久化的，那么确认消息会在消息写入

磁盘之后发出。`RabbitMQ`回传给生产者的确认消息中的`deliveryTag`包含了确认消息的序号，此外`RabbitMQ`也可以设置`channel.basicAck`方法中的`multiple`参

数，表示到这个序号之前的所有消息都已经得到了处理。

![](image/QQ截图20210727084816.png)

​		发送方确认机制最大的好处在于它是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认

之后，生产者应用程序便可以通过回调方法来处理该确认消息，如果`RabbitMQ`因为自身内部错误导致消息丢失，就会发送一条`nack`命令，生产者应用程序同样可

以在回调方法中处理该`nack`命令。

​		生产者通过调用`channel.confirmSelect`方法将信道设置为`confirm`模式，之后`RabbitMQ`会返回命令表示同意生产者将当前信道设置为`confirm`模式。所有

被发送的后续消息都被`ack`或者`nack`一次，不会出现一条消息既被`ack`又被`nack`的情况，并且`RabbitMQ`也并没有对消息被`confirm`的快慢做任何保证。

```java
		try
        {
            channel.confirmSelect();
            channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes()); // 发送一条持久化消息
            if(!channel.waitForConfirms())
            {
                System.out.println("send message failed");
            }

        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
```

​		发送多条消息：

```java
		for(int i = 0 ; i < length ; i++)
        {
            try
            {
                channel.confirmSelect(); // 这里与事务机制不同
                channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes()); // 发送一条持久化消息
                if(!channel.waitForConfirms()) // 此种方式与事务机制相同
                {
                    System.out.println("send message failed");
                }

            }
            catch (Exception e)
            {
                e.printStackTrace();
            }
        }
```

![](image/QQ截图20210727091537.png)

```java
boolean waitForConfirms() throws InterruptedException;
boolean waitForConfirms(long timeout) throws InterruptedException, TimeoutException;
void waitForConfirmsOrDie() throws IOException, InterruptedException;
void waitForConfirmsOrDie(long timeout) throws IOException, InterruptedException, TimeoutException;
```

​		如果信道没有开启`confirm`模式，则调用任何`waitForConfirms`方法都会报出`java.lang.IllegalStateException`。对于没有参数的`waitForConfirms`方法来

说，其返回的条件是客户端收到了相应的确认或拒绝命令或者被中断。参数`timeout`表示超时时间，一旦等待`RabbitMQ`回应超时就会抛出

`java.util.concurrent.TimeoutException`的异常。两个`waitForConfirmsorDie`方法在接收到`RabbitMQ`返回的拒绝命令之后会抛出`java.io.IOException`。

​		

​		确认模式的优势在于并不一定需要同步确认：

​				1、批量确认：每发送一批消息后，调用`channel.waitForConfirms`方法，等待服务器的返回确认。

```java
		try
        {
            channel.confirmSelect();
            int count = 0;
            while (true)
            {
                channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes()); // 发送一条持久化消息
                // 将发送出去的消息存入缓存
                if(++count >= BATCH_COUNT)
                {
                    count = 0;
                    try
                    {
                        if(channel.waitForConfirms())
                        {
                            // 将缓存中的消息清空
                        }
                        // 将缓存中的消息重新发送
                    }
                    catch (Exception e)
                    {
                        e.printStackTrace();
                        // 将缓存中的消息重新发送
                    }
                }
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
```



​				2、异步确认方法：提供一个回调方法，服务器确认了一条或者多条消息后服务器会回调这个方法进行处理。在客户端`Channel`接口中提供的

`addConfirmListener`方法可以添加`ConfirmListener`这个回调接口，这个`ConfirmListener`接口包含两个方法：`handleAck`和`handleNack`，分别用来处理

`RabbitMQ`回传的`Basic.Ack`和`Basic.Nack`。在这两个方法中都包含有一个参数`deliveryTag(`在`confirm`模式下用来标记消息的唯一有序序号)。需要为每一个信

道维护一个`"unconfirm"(`采用有序集合`SortedSet)`的消息序号集合，每发送一条消息，集合中的元素加`1`。每当调用`ConfirmListener`中的`handleAck`方法

时，`unconfirm`集合中删掉相应的一条`(multiple`设置为`false)`或者多条`(multiple`设置为`true)`记录。

```java
		SortedSet<Long> confirmSet = new TreeSet(); // 消息序号集合
        channel.confirmSelect();
        channel.addConfirmListener(new ConfirmListener() {
            @Override
            public void handleAck(long deliveryTag, boolean multiple) throws IOException {
                System.out.println("ack,seqNo:" + deliveryTag + ", multiple：" + multiple);
                if(multiple)
                {
                    confirmSet.headSet(deliveryTag - 1).clear();
                }
                else
                {
                    confirmSet.remove(deliveryTag);
                }
            }

            @Override
            public void handleNack(long deliveryTag, boolean multiple) throws IOException {
                if(multiple)
                {
                    confirmSet.headSet(deliveryTag - 1).clear();
                }
                else
                {
                    confirmSet.remove(deliveryTag)
                }
                // 这里需要添加处理消息重发的场景
            }
        });
		
		// 模拟一致发送消息
        while (true)
        {
            long no = channel.getNextPublishSeqNo();
            channel.basicPublish(EXCHANGE_NAME,ROUTING_KEY, MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes()); // 发送一条持久化消息
            confirmSet.add(no);
        }
```

​		事务机制和`confirm`机制确保的是消息能够正确地发送至`RabbitMQ`，指消息被正确地发往至`RabbitMQ`的交换器，如果此交换器没有匹配的队列，那么消息也

会丢失。所以在使用这两种机制的时候要确保所涉及的交换器能够有匹配的队列。更进一步地讲，发送方要配合`mandatory`参数或者备份交换器一起使用来提高消

息传输的可靠性。



## 消费端

### 消息分发

​		当`RabbitMQ`队列拥有多个消费者时，队列收到的消息将以轮询的分发方式发送给消费者。每条消息只会发送给订阅列表里的一个消费者。而且它是专门为并

发程序设计的。如果现在负载加重，那么只需要创建更多的消费者来消费处理消息即可。

​		但此种方式有时并不是很优雅，比如当某些消费者任务繁重，来不及消费消息，则会造成性能的下降。`channel.basicQos`方法允许限制信道上的消费者所能

保持的最大未确认消息的数量。`RabbitMQ`会保存一个消费者的列表，每发送一条消息都会为对应的消费者计数，如果达到了所设定的上限，那么`RabbitMQ`就不会

向这个消费者再发送任何消息。直到消费者确认了某条消息之后，`RabbitMQ`将相应的计数减1，之后消费者可以继续接收消息，直到再次到达计数上限。这种机制

可以类比于`TCP/IP`中的滑动窗口。但此种方式只对于推模式有效，对于拉模式是无效的。

```java
void basicQos(int prefetchCount) throws IOException;
void basicQos(int prefetchCount, boolean global) throws IOException;
void basicQos(int prefetchSize, int prefetchCount, boolean global) throws IOException;

/*
prefetchCount：最大未确认消息的数量，0表示没有上限
prefetchSize：总体大小的上限，0表示没有上限
global：默认为false
*/

```

![](image/QQ截图20210727104226.png)



### 消息顺序性

​		消息的顺序性是指消费者消费到的消息和发送者发布的消息的顺序是一致的。可以通过在消息体内添加全局有序标识来实现。



### 消息传输保障

​		消息中间件的消息传输保障分为三个层级：`RabbitMQ`支持最多一次和最少一次

​				1、最多一次：消息可能会丢失，但绝不会重复传输。

​				2、最少一次：消息绝不会丢失，但可能会重复传输。

​				3、恰好一次：每条消息肯定会被传输一次且仅传输一次。

​		最少一次需要考虑的内容：

​				1、消息生产者需要开启事务机制或者`confirm`机制，以确保消息可以可靠地传输到`RabbitMQ`中。

​				2、消息生产者需要配合使用`mandatory`参数或者备份交换器来确保消息能够从交换器路由到队列中，进而能够保存下来而不会被丢弃。

​				3、消息和队列都需要进行持久化处理，以确保`RabbitMQ`服务器在遇到异常情况时不会造成消息丢失。

​				4、消费者在消费消息的同时需要将`autoAck`设置为`false`，然后通过手动确认的方式去确认已经正确消费的消息，以避免在消费端引起不必要的消息丢

​		失。

​		最多一次的方式就无须考虑以上那些方面，生产者随意发送，消费者随意消费，不过这样很难确保消息不会丢失。

​		目前大多数主流的消息中间件都没有消息去重机制，都不保障恰好一次。去重处理一般是在业务客户端实现，比如引入`GUID(Globally Unique Identifier)`的

概念。针对`GUID`，如果从客户端的角度去重，那么需要引入集中式缓存，必然会增加依赖复杂度，另外缓存的大小也难以界定。建议在实际生产环境中，业务方

根据自身的业务特性进行去重，比如业务消息本身具备幂等性，或者借助Redis等其他产品进行去重处理。



# 管理

## 虚拟主机

​		每一个`RabbitMQ`服务器都能创建虚拟的消息服务器，我们称之为虚拟主机，简称为`vhost`。每一个`vhost`本质上是一个独立的小型`RabbitMQ`服务器，拥有自

己独立的队列交换器及绑定关系等，并且它拥有自己独立的权限。`vhost`就像是虚拟机与物理服务器一样，它们在各个实例间提供逻辑上的分离，为不同程序安全

保密地运行数据，它既能将同一个`RabbitMQ`中的众多客户区分开，又可以避免队列和交换器等命名冲突。`vhost`之间是绝对隔离的，无法将不用`vhost`之间的交

换器与队列进行绑定，这样既保证了安全性，又可以确保可移植性。默认的`vhost`为`/`。

​		创建`vhost`：

```shell
rabbitmqctl add_vhost host_name
```

​		查看：

```shell
rabbitmqctl list_vhost [vhostinfoitem]

vhostinfoitem有两个取值：
	name:表示vhost的名称
	tracing：是否使用了RabbitMQ的trace功能
```

​		删除：删除一个`vhost`同时也会删除其下所有的队列、交换器、绑定关系、用户权限、参数和策略等信息

```shell
rabbitmqctl delete_vhost host_name
```



​		在`RabbitMQ`中，权限控制是以`vhost`为单位的，当创建一个用户时，用户通常会被指派给至少一个`vhost`，并且只能访问被指派的`vhost`内的队列、交换器

和绑定关系等。所以`RabbitMQ`中的赋予权限是指在`vhost`级别对用户而言的权限授予。

```shell
// 配置权限
rabbitmqctl set_permissions [-p vhost] {user} {conf} {write} {read}
	vhost:授予用户访问权限的vhost名称，默认值为"/"
	user:可以访问指定vhost的用户名
	conf:一个用于匹配用户在哪些资源上拥有可配置(队列和交换器的创建和删除之类的操作)权限的正则表达式
	write:一个用于匹配用户在哪些资源上拥有可写(发布消息)权限的正则表达式
	read:一个用于匹配用户在哪些资源上拥有可读(与消息有关的操作，包括读取消息及清空整个队列等)权限的正则表达式
	
// 清除权限
rabbitmqctl clear_permissions [-p vhost] {username}
```

​		

​		两个命令查看权限权限信息：

```shell
rabbitmqctl list_permissions [-p vhost]：显示虚拟主机的权限
rabbitmqctl list_user_permissions {username}：显示用户的权限
```



## 用户

​		在`RabbitMQ`中，用户是访问控制的基本单元，且单个用户可以跨越多个`vhost`进行授权。针对一至多个`vhost`，用户可以被赋予不同级别的访问权限，并使

用标准的用户名和密码来认证用户。

​		创建：

```shell
rabbitmqctl add_user {username} {password}
```

​		修改密码：

```shell
rabbitmqctl change_password {username} {newpassword}
```

​		清除密码：

```shell
rabbitmqctl clear_password {username}
```

​		验证用户：

```shell
rabbitmqctl authenticate_user {username} {password}
```

​		删除用户：

```shell
rabbitmqctl delete_user {username}
```

​		查看用户：

```shell
rabbitmqctl list_users
```



​		`5`中角色：

​				`none`：无任何角色。新创建的用户的角色默认为`none`。

​				`management`：可以访问`Web`管理页面。

​				`policymaker`：包含`management`的所有权限，并且可以管理策略和参数。

​				`monitoring`：包含`management`的所有权限，并且可以看到所有连接、信道及节点相关的信息。

​				`administartor`：包含`monitoring`的所有权限，并且可以管理用户、虚拟主机、权限、策略、参数等。`administator`代表了最高的权限。

​		设置角色：

```shell
rabbitmqctl set_user_tags {username} {tag ...}
	tag：用于设置0或多个角色，设置之后任何之前现有的身份都会被删除
```



## Web端管理

​		`RabbitMQ management`插件可以提供`Web`管理界面用来管理如前面所述的虚拟主机、用户等，也可以用来管理队列、交换器、绑定关系、策略、参数等，还可

以用来监控`RabbitMQ`服务的状态及一些数据统计类信息，可谓是功能强大，基本上能够涵盖所有`RabbitMQ`管理的功能。

​		插件存放在`$RABBITMQ_HOME/plugins`目录下。

​		启动插件：

```shell
rabbitmq-plugins enable [plugin-name]
```

​		关闭插件：

```shell
rabbitmq-plugins disable [plugin-name]
```

​		查看插件使用情况：

```shell
rabbitmq-plugins list
	[E*]：显式启动
	[e*]：隐式启动
```



​		`HTTP API`：不仅可以使用`Web`界面来管理`RabbitMQ`，也可以使用其提供的`API`来实现，完全基于`RESTful`风格。可以通过`Web`页面上的`HTTP API`链接来查

看所有的`API`。



# 配置

## 环境变量

​		`RabbitMQ`的环境变量都是以`RABBITMQ_`开头的，可以在`Shell`环境中设置，也可以在`rabbitmq-env.conf(`默认在`$RABBITMQ/etc/rabbitmq/`目录下，可以在

启动时指定`RABBITMQ_CONF_ENV_FILE`变量来设置此文件的路径`)`这个`RabbitMQ`环境变量的定义文件中设置。如果是在非`Shell`环境中配置，则需要将`RABBITMQ_`

这个前缀去除。优先级顺序按照`Shell`环境最优先，其次`rabbitmg-env.conf`配置文件，最后是默认的配置。

​		如果需要制定节点的名称，而不是采用默认的方式，可以在`rabbitmq-server`命令前添加`RABBITMO NODENAME`变量来设定指定的名称。



## 参数和策略

​		`Parameter`可以通过`rabbitmqctl`工具或者`RabbitMQ Management`插件提供的`HTTP API`接口来设置。`RabbitMQ`中一共有两种类型的`Parameter`：`vhost`级别

的`Parameter`和`global`级别的`Parameter`。`vhost`级别的`Parameter`由一个组件名称、名称和值组成，而`global`级别的参数由一个名称和值组成，不管是

`vhost`级别还是`global`级别的参数，其所对应的值都是`JSON`类型的。

​		策略`(Policy)`是一种特殊的参数，其是`vhost`级别的。一个`Policy`可以匹配一个或者多个队列`(`或者交换器，或者两者兼有`)`，这样便于批量管理。与此

同时，`Policy`也可以支持动态地修改一些属性参数，大大地提高了应用的灵活度。一般来说，`Policy`用来配置`Federation`、镜像、备份交换器、死信等功能。

​		如果有多个`Policy`作用于同一个交换器或者队列，那么`Priority`最大的那个`Policy`才会有用。如果两个或多个`Policy`都作用到同一个交换器或者队列上，

且这些`Policy`的优先级都是一样的，则参数项最多的`Policy`具有决定权。如果参数一样多，则最后添加的`Policy`具有决定权。



# 集群

​		`RabbitMQ`中的每一个节点，不管是单一节点系统或者是集群中的一部分，要么是内存节点，要么是磁盘节点。内存节点将所有的队列、交换器、绑定关系、

用户、权限和`vhost`的元数据定义都存储在内存中，而磁盘节点则将这些信息存储到磁盘中。单节点的集群中必然只有磁盘类型的节点，否则当重启`RabbitMQ`之

后，所有关于系统的配置信息都会丢失。不过在集群中，可以选择配置部分节点为内存节点，这样可以获得更高的性能。

​		如果集群中唯一的磁盘节点崩溃，集群仍然可以保持运行`(`发送和接收消息`)`，但是直到将该节点恢复到集群前，无法更改任何东西。

​		`RabbitMQ`的日志默认存放在`$RABBITMQ_HOME/var/log/rabbitmq`文件夹内。在这个文件夹内`RabbitMQ`会创建两个日志文件：`RABBITMO_NODENAME-sasl.log`和

`RABB工TMQ_NODENAME.log`。前者是关于`Erlang`的日志，后者才是`RabbitMQ`的服务日志。

​		`RabbitMQ`默认会创建一些交换器，其中`amq.rabbitmq.log`就是用来收集`RabbitMQ`日志的，集群中所有的服务日志都会发往这个交换器中。这个交换器的类

型为`topic`，可以收集`debug、info、warning`和`error`这`4`个级别的日志。



# 迁移

​		`RabbitMQ`中的集群迁移更多的是用来解决集群故障不可短时间内修复而将所有的数据、客户端连接等迁移到新的集群中，以确保服务的可用性。相比于单点

故障而言，集群故障的危害性就大得多。这时候就需要通过集群迁移重新建立起一个新的集群。

​		`RabbitMQ`集群迁移包括元数据重建、数据迁移，以及与客户端连接的切换。

​		元数据重建是指在新的集群中创建原集群的队列、交换器、绑定关系、`vhost`、用户、权限和`Parameter`等数据信息。元数据重建之后才可将原集群中的消息

及客户端连接迁移过来。

​		数据迁移的主要原理是先从原集群中将数据消费出来，然后存入一个缓存区中，另一个线程读取缓存区中的消息再发布到新的集群中，如此便完成了数据迁移

的动作。



# Federation

​		`Federation`插件的设计目标是使`RabbitMQ`在不同的`Broker`节点之间进行消息传递而无须建立集群，该功能在很多场景下都非常有用：

​				1、`Federation`插件能够在不同管理域`(`可能设置了不同的用户和`vhost`，也可能运行在不同版本的`RabbitMQ`和`Erlang`上`)`中的`Broker`或者集群之间

​		传递消息。

​				2、`Federation`插件基于`AMQP 0-9-1`协议在不同的`Broker`之间进行通信，并设计成能够容忍不稳定的网络连接情况。

​				3、一个`Broker`节点中可以同时存在联邦交换器`(`或队列`)`或者本地交换器`(`或队列`)`，只需要对特定的交换器`(`或队列`)`创建`Federation`连接。

​				4、`Federation`不需要在N个`Broker`节点之间创建`O(N²)`个连接`(`尽管这是最简单的使用方式`)`，这也就意味着`Federation`在使用时更容易扩展。

​		`Federation`插件可以让多个交换器或者多个队列进行联邦。一个联邦交换器或者一个联邦队列接收上游的消息，上游是指：位于其他`Broker`上的交换器或者

队列。联邦交换器能够将原本发送给上游交换器的消息路由到本地的某个队列中；联邦队列则允许一个本地消费者接收到来自上游队列的消息。

​		除了联邦交换器，`RabbitMQ`还可以支持联邦队列。联邦队列可以在多个`Broker`节点`(`或者集群`)`之间为单个队列提供均衡负载的功能。一个联邦队列可以

连接一个或者多个上游队列，并从这些上游队列中获取消息以满足本地消费者消费消息的需求。

# Shovel

​		与`Federation`具备的数据转发功能类似，`Shovel`能够可靠、持续地从一个`Broker`中的队列或交换器`(`作为源端，即`source)`拉取数据并转发至另一个

`Broker`中的交换器或队列`(`作为目的端，即`destination)`。作为源端的队列或交换器和作为目的端的交换器或队列可以同时位于同一个`Broker`上，也可以位于

不同的`Broker`上。

​		主要优势在于：

​				1、松耦合。`Shovel`可以移动位于不同管理域中的`Broker(`或者集群`)`上的消息，这些`Broker(`或者集群`)`可以包含不同的用户和`vhost`，也可以使用

​		不同的`RabbitMQ`和`Erlang`版本。

​				2、支持广域网。`Shovel`插件同样基于`AMQP`协议在`Broker`之间进行通信，被设计成可以容忍时断时续的连通情形，并且能够保证消息的可靠性。

​				3、高度定制。当`Shovel`成功连接后，可以对其进行配置以执行相关的`AMQP`命令。



# 实现原理

## 存储机制

​		不管是持久化的消息还是非持久化的消息都可以被写入到磁盘。持久化的消息在到达队列时就被写入到磁盘，并且如果可以，持久化的消息也会在内存中保存

一份备份，这样可以提高一定的性能，当内存吃紧的时候会从内存中清除。非持久化的消息一般只保存在内存中，在内存吃紧的时候会被换入到磁盘中，以节省内

存空间。这两种类型的消息的落盘处理都在`RabbitMQ`的持久层中完成。

​		持久层是一个逻辑上的概念，实际包含两个部分：

​				1、队列索引`(rabbit_queue_index)`：`rabbit_queue_index`负责维护队列中落盘消息的信息，包括消息的存储地点、是否已被交付给消费者、是否已被

​		消费者确认等。每个队列都有与之对应的一个`rabbit_queue_index`。

​				2、消息存储`(rabbit_msg_store)`：`rabbit_msg_store`以键值对的形式存储消息，它被所有队列共享，在每个节点中有且只有一个。从技术层面上来

​		说，`rabbit_msg_store`具体还可以分为`msg_store_persistent`和`msg _store_transient`，`msg_store _persistent`负责持久化消息的持久化，重启后消息不

​		会丢失；`msg_store_transient`负责非持久化消息的持久化，重启后消息会丢失。通常情况下，习惯性地将`msg_store_persistent`和`msg_store_transient`看

​		成`rabbit_msg_store`这样一个整体。

​		

​		消息`(`包括消息体、属性和`headers)`可以直接存储在`rabbit_queue_index`中，也可以被保存在`rabbit_msg_store`中。默认在

`$RABBITMQ_HOME/var/lib/mnesia/rabbite$HOSTNAME/`路径下包含`queues、msg_store_persistent、msg_store_transient`这3个文件夹，其分别存储对应的信息。默

认小于`2k(`该值可以通过`queue_index_embed_msgs_below`来配置`)`的消息`(`包括消息体，属性及`headers`整体的大小`)`存储在队列索引中，大于`2k`的存储在消息

存储中。

​		队列索引中以顺序`(`文件名从`0`开始累加`)`的段文件来进行存储，后缀为`".idx"`，每个段文件中包含固定的`SEGMENT_ENTRY_COUNT`条记录

`SEGMENT_ENTRY_COUNT`默认值为`16384`。每个队列索引中从磁盘中读取消息的时候至少要在内存中维护一个段文件，所以设置`queue_index_embed_msgs _below`值

的时候要格外谨慎，一点点增大也可能会引起内存爆炸式的增长。

​		经过消息存储处理的所有消息都会以追加的方式写入到文件中，当一个文件的大小超过指定的限制`(file_size_limit)`后，关闭这个文件再创建一个新的文件

以供新的消息写入。文件名`(`后缀是`". rdq")`从`0`开始进行累加，因此文件名最小的文件也是最老的文件。在进行消息的存储时，`RabbitMQ`会在`ETS`表中记录消

息在文件中的位置映射和文件的相关信息。在读取消息的时候，先根据消息的`ID`找到对应存储的文件，如果文件存在并且未被锁住，则直接打开文件，从指定位

置读取消息的内容。如果文件不存在或者被锁住了，则发送请求由消息存储进行处理。消息的删除只是从`ETS`表删除指定消息的相关信息，同时更新消息对应的存

储文件的相关信息。执行消息删除操作时，并不立即对在文件中的消息进行删除，也就是说消息依然在文件中，仅仅是标记为垃圾数据而已。当一个文件中都是垃

圾数据时可以将这个文件删除。当检测到前后两个文件中的有效数据可以合并在一个文件中，并且所有的垃圾数据的大小和所有文件`(`至少有`3`个文件存在的情况

下`)`的数据大小的比值超过设置的阈值`GARBAGE_FRACTION(`默认值为`0.5)`时才会触发垃圾回收将两个文件合并。

​		执行合并的两个文件一定是逻辑上相邻的两个文件。执行合并时首先锁定这两个文件，并先对前面文件中的有效数据进行整理，再将后面文件的有效数据写入

到前面的文件，同时更新消息在`ETS`表中的记录，最后删除后面的文件。

![](image/QQ截图20210804071531.png)



### 队列的结构

​		通常队列由`rabbit_amgqueue_process`和`backing_queue`这两部分组成，`rabbit_amgqueue_process`负责协议相关的消息处理，即接收生产者发布的消息、向

消费者交付消息、处理消息的确认`(`包括生产端的`confirm`和消费端的`ack)`等。`backing_queue`是消息存储的具体形式和引擎，并向`rabbit_amqqueue_process`提

供相关的接口以供调用。

​		如果消息投递的目的队列是空的，并且有消费者订阅了这个队列，那么该消息会直接发送给消费者，不会经过队列这一步。而当消息无法直接投递给消费者

时，需要暂时将消息存入队列，以便重新投递。

​		队列消息的状态：

​				`alpha`：消息内容`(`包括消息体、属性和`headers)`和消息索引都存储在内存中。此状态最耗内存，但很少消耗`CPU`。

​				`beta`：消息内容保存在磁盘中，消息索引保存在内存中。此状态需要进行一次`I/O`操作才能读到消息。

​				`gamma`：消息内容保存在磁盘中，消息索引在磁盘和内存中都有。该状态只有持久化的消息才会有。此状态需要进行一次`I/O`操作才能读到消息。对于

​		`durable`属性设置为`true`的消息，它一定会进入此状态，并且在开启生产者确认机制时，只有到了此状态时才会确认该消息已被接收。

​				`delta`：消息内容和索引都在磁盘中。此状态需要进行两次`I/O`操作才能读到消息。此状态基本不消耗内存，但需要消耗更多的`CPU`和磁盘`I/O`操作。

​		`RabbitMQ`在运行时会根据统计的消息传送速度定期计算一个当前内存中能够保存的最大消息数量，如果`alpha`状态的消息数量大于此值时，就会引起消息的

状态转换，多余的消息可能会转换到`beta`状态、`gamma`状态或者`delta`状态。

​		对于普通的没有设置优先级和镜像的队列来说，`backing _queue`的默认实现是`rabbit_variable_queue`，其内部通过`5`个子队列`Q1、Q2、Delta、Q3`和`Q4`来

体现消息的各个状态。整个队列包括`rabbit_amqqueue_process`和`backing _queue`的各个子队列。

![](image/QQ截图20210804074133.png)

​		`Q1、Q4`只包含`alpha`状态的消息，`Q2`和`Q3`包含`beta`和`gamma`状态的消息，`Delta`只包含`delta`状态的消息。一般情况下，消息按照

`Q1→Q2→Delta→Q3→Q4`这样的顺序步骤进行流动，但并不是每一条消息都一定会经历所有的状态，这个取决于当前系统的负载状况。从`Q1`至`Q4`基本经历内存到磁

盘，再由磁盘到内存这样的一个过程，如此可以在队列负载很高的情况下，能够通过将一部分消息由磁盘保存来节省内存空间，而在负载降低的时候，这部分消息

又渐渐回到内存被消费者获取，使得整个队列具有很好的弹性。

​		消费者获取消息也会引起消息的状态转换。当消费者获取消息时，首先会从`Q4`中获取消息，如果获取成功则返回。如果`Q4`为空，则尝试从`Q3`中获取消息，

系统首先会判断`Q3`是否为空，如果为空则返回队列为空，即此时队列中无消息。如果`Q3`不为空，则取出`Q3`中的消息，进而再判断此时`Q3`和`Delta`中的长度，

如果都为空，则可以认为`Q2、Delta、Q3、Q4`全部为空，此时将`Q1`中的消息直接转移至`Q4`，下次直接从`Q4`中获取消息。如果`Q3`为空，`Delta`不为空，则将

`Delta`的消息转移至`Q3`中，下次可以直接从`Q3`中获取消息。在将消息从`Delta`转移到`Q3`的过程中，是按照索引分段读取的，首先读取某一段，然后判断读取

的消息的个数与`Delta`中消息的个数是否相等，如果相等，则可以判定此时`Delta`中已无消息，则直接将`Q2`和刚读取到的消息一并放入到`Q3`中；如果不相等，仅

将此次读取到的消息转移到`Q3`。



### 惰性队列

​		队列具备两种模式：`default`和`lazy`。默认的为`default`模式。`lazy`模式即为惰性队列的模式，可以通过调用`channel.queueDeclare`方法的时候在参数

`(x-queue-mode)`中设置，也可以通过`Policy`的方式设置。

​		惰性队列会尽可能地将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够支持更长的队列，即支持更多

的消息存储。

​		当生产者将消息发送到`RabbitMQ`的时候，队列中的消息会尽可能地存储在内存之中，这样可以更加快速地将消息发送给消费者。即使是持久化的消息，在被

写入磁盘的同时也会在内存中驻留一份备份。当RabbitMQ需要释放内存的时候，会将内存中的消息换页至磁盘中，这个操作会耗费较长的时间，也会阻塞队列的

操作，进而无法接收新的消息。

​		惰性队列会将接收到的消息直接存入文件系统中，而不管是持久化的或者是非持久化的这样可以减少了内存的消耗，但是会增加`I/O`的使用，如果消息是持

久化的，那么这样的`I/O`操作不可避免。如果惰性队列中存储的是非持久化的消息，内存的使用率会一直很稳定，但是重启之后消息一样会丢失。



## 内存及磁盘预警

​		当内存使用超过配置的阈值或者磁盘剩余空间低于配置的阈值时，`RabbitMQ`都会暂时阻塞客户端的连接并停止接收从客户端发来的消息，以此避免服务崩

溃。与此同时，客户端与服务端的心跳检测也会失效。可以通过`rabbitmqctl list_connections`命令或者`Web`管理界面来查看它的状态。如果一个服务节点的内存

或者磁盘受限，都会引起整个集群中所有的连接被阻塞。

​		`RabbitMQ`服务器会在启动或者执行`rabbitmqctl set_vm_memory_high_watermarkfraction(`重启会失效`)`命令时计算系统内存的大小。默认情况下

`vm_memory_high_watermark`的值为`0.4`，即内存阈值为`0.4`，表示当`RabbitMQ`使用的内存超过`40%`时，就会产生内存告警并阻塞所有生产者的连接并尝试将队列

中的消息`(`包括持久化和非持久化`)`换页到磁盘。一旦告警被解除`(`有消息被消费或者从内存转储到磁盘等情况的发生`)`，一切都会恢复正常。如果设置为`0`，

所有生产者都会被停止发送消息，可以适用于禁止集群中所有消息发布的情况。

​		默认情况下，在内存到达内存阈值的`50%`时会进行换页动作。也就是说，在默认的内存阈值为`0.4`的情况下，当内存超过`0.4×0.5=0.2`时会进行换页动作。

可以通过在配置文件中配置`vm_memory_high_watermark_paging_ratio`项来修改此值。若此值大于`1`，表示禁用换页功能。

​		当剩余磁盘空间低于确定的阈值时，`RabbitMQ`同样会阻塞生产者，这样可以避免因非持久化的消息持续换页而耗尽磁盘空间导致服务崩溃。默认情况下，磁

盘阈值为`50MB`，这意味着当磁盘剩余空间低于`50MB`时会阻塞生产者并停止内存中消息的换页动作。这个阈值的设置可以减小但不能完全消除因磁盘耗尽而导致崩

溃的可能性。



## 流控

​		流控机制是用来避免消息的发送速率过快而导致服务器难以支撑的情形。内存和磁盘告警相当于全局的流控，一旦触发会阻塞集群中所有的连接，而此处的流

控是针对单个连接的。

​		`Erlang`进程之间并不共享内存，而是通过消息传递来通信，每个进程都有自己的进程邮箱。`RabbitMQ`使用了一种基于信用证算法的流控机制来限制发送消息

的速率。它通过监控各个进程的进程邮箱，当某个进程负载过高而来不及处理消息时，这个进程的进程邮箱就会开始堆积消息。当堆积到一定量时，就会阻塞而不

接收上游的新消息。从而慢慢地，上游进程的进程邮箱也会开始堆积消息。当堆积到一定量时也会阻塞而停止接收上游的消息，最后就会使负责网络数据包接收的

进程阻塞而暂停接收新的数据。

​		流控机制不只是作用于连接，同样作用于信道和队列。从连接到信道，再到队列，最后是消息持久化存储形成一个完整的流控链，对于处于整个流控链中的任

意进程，只要该进程阻塞，上游的进程必定全部被阻塞。也就是说，如果某个进程达到性能瓶颈，必然会导致上游所有的进程被阻塞。

![](image/QQ截图20210804083358.png)

​		`rabbit_reader`：连接的处理进程，负责接收、解析AMQP协议数据包等。

​		`rabbit_channel`：`Channel`的处理进程，负责处理`AMQP`协议的各种方法、进行路由解析等。

​		`rabbit_amgqueue_process`：队列的处理进程，负责实现队列的所有逻辑。

​		`rabbit_msg _store`：负责实现消息的持久化。



# 镜像队列

​		引入镜像队列的机制，可以将队列镜像到集群中的其他服务节点之上如果集群中的一个节点失效了，队列能自动地切换到镜像中的另一个节点上以保证服务的

可用性。在通常的用法中，针对每一个配置镜像的队列都包含一个主节点和若干个从节点。

![](image/QQ截图20210804084307.png)

​		发送消息到镜像队列时，会将消息同时发往主节点和各个从节点。除发送消息外的所有动作都只会向主节点发送，在由主节点将命令执行的结果广播到各个从

节点。

​		镜像队列同时支持生产者和事务两种机制。在事务机制中，只有当前事务在全部镜像中执行之后，客户端才会收到提交成功的消息。同样的，在生产者确认机

制中，生产者进行当前消息确认的前提是该消息被全部进行所接收了。

​		镜像队列的`backing _queue`比较特殊，其实现并非是`rabbit_variable_queue`，它内部包裹了普通`backing_queue`进行本地消息消息持久化处理，在此基础上

增加了将消息和确认复制到所有镜像的功能。

![](image/QQ截图20210804085059.png)



# 网络分区

​		`RabbitMQ`集群的网络分区的容错性并不是很高，一般都是使用`Federation`或者`Shovel`来解决广域网中的使用问题。不过即使是在局域网环境下，网络分区

也不可能完全避免，网络设备出现故障也会导致网络分区。当出现网络分区时，不同分区里的节点会认为不属于自身所在分区的节点都已经挂了，对于队列、交换

器、绑定的操作仅对当前分区有效。在`RabbitMQ`的默认配置下，即使网络恢复了也不会自动处理网络分区带来的问题。

​		当一个集群发生网络分区时，这个集群会分成两个部分或者更多，它们各自为政，互相都认为对方分区内的节点已经挂了，包括队列、交换器及绑定等元数据

的创建和销毁都处于自身分区内，与其他分区无关。如果原集群中配置了镜像队列，而这个镜像队列又牵涉两个或者更多个网络分区中的节点时，每一个网络分区

中都会出现一个`master`节点，对于各个网络分区，此队列都是相互独立的。

​		`RabbitMQ`集群节点内部通信端口默认为`25672`，两两节点之间都会有信息交互。如果某节点出现网络故障，或者是端口不通，则会致使与此节点的交互出现

中断，这里就会有个超时判定机制，继而判定网络分区。对于网络分区的判定是与`net_ticktime`这个参数息息相关的，此参数默认值为60秒。在`RabbitMQ`集群内

部的每个节点之间会每隔四分之一的`net_ticktime`计一次应答。如果有任何数据被写入节点中，则此节点被认为已经被应答了。如果连续`4`次，某节点都没有被

应答，则可以判定此节点已处于`down`状态，其余节点可以将此节点剥离出当前分区。



​		对于未配置镜像的集群，网络分区发生之后，队列也会伴随着宿主节点而分散在各自的分区之中。对于消息发送方而言，可以成功发送消息，但是会有路由失

败的现象，需要配合`mandatory`等机制保障消息的可靠性。对于消息消费方来说，有可能会有诡异、不可预知的现象发生，比如对于已消费消息的确认会失效。如

果网络分区发生之后，客户端与某分区重新建立通信链路，其分区中如果没有相应的队列进程，则会有异常报出。如果从网络分区中恢复之后，数据不会丢失，但

是客户端会重复消费。

​		网络分区的发生可能会引起消息的丢失，当然这点也有办法解决。首先消息发送端要有能够处理`Basic.Return`的能力。其次，在监测到网络分区发生之后，

需要迅速地挂起所有的生产者进程。之后连接分区中的每个节点消费分区中所有的队列数据。在消费完之后再处理网络分区。最后在从网络分区中恢复之后再恢复

生产者的进程。整个过程可以最大程度上保证网络分区之后的消息的可靠性。同样也要注意的是，在整个过程中会伴有大量的消息重复，消费者客户端需要做好相

应的幂等性处理。当然也可以采用集群迁移，将所有旧集群的资源都迁移到新集群来解决这个问题。

​		为了从网络分区中恢复，首先需要挑选一个信任分区，在挑选完信任分区之后，重启非信任分区中的节点，如果此时还有网络分区的告警，紧接着重启信任分

区中的节点。

​		`RabbitMQ`的重启顺序也比较讲究，必须在以下两种重启顺序中择其一进行重启操作，同时如果配置了镜像队列会产生队列漂移的现象，可以在重启之前删除

镜像队列的配置，在一定程度上阻止这种现象的发生：

​				1、停止其他非信任分区中的所有节点，然后再启动这些节点。如果此时还有网络分区的告警，则再重启信任分区中的节点以去除告警。

​				2、关闭整个集群中的节点，然后再启动每一个节点，这里需要确保启动的第一个节点在信任的分区之中。

​		具体的网络分区处理步骤：

​				1、挂起生产者和消费者进程。这样可以减少消息不必要的丢失,如果进程数过多，情形又比较紧急，也可跳过此步骤。

​				2、删除镜像队列的配置。

​				3、挑选信任分区。

​				4、关闭非信任分区中的节点。采用`rabbitmqctl stop_app`命令关闭。

​				5、启动非信任分区中的节点。采用与步骤`4`对应的`rabbitmqctl start_app`命令启动。

​				6、检查网络分区是否恢复，如果已经恢复则转步骤`8`；如果还有网络分区的报警则转步骤`7`。

​				7、重启信任分区中的节点。

​				8、添加镜像队列的配置。

​				9、恢复生产者和消费者的进程。



​		`RabbitMQ`提供了三种方法自动地处理网络分区：`pause-minority`模式、`pause-if-all-down`模式和`autoheal`模式。默认是`ignore`模式，即不自动处理网络

分区，所以在这种模式下，当网络分区的时候需要人工介入。



​		在`pause-minority`模式下，当发生网络分区时，集群中的节点在观察到某些节点`down`的时候，会自动检测其自身是否处于少数派`(`分区中的节点小于或者等

于集群中一半的节点数`)`，`RabbitMQ`会自动关闭这些节点的运作。这样确保了在发生网络分区的情况下，大多数节点`(`当然这些节点得在同一个分区中`)`可以继

续运行。少数派中的节点在分区开始时会关闭，当分区结束时又会启动。这里关闭是指`RabbitMQ`应用的关闭，而`Erlang`虚拟机并不关闭，类似于执行了

`rabbitmqctl stop_app`命令。处于关闭的节点会每秒检测一次是否可连通到剩余集群中,如果可以则启动自身的应用。相当于执行`rabbitmqctl start_app`命令。

​		当对等分区出现时，会关闭这些分区内的所有节点，只有等待网络恢复之后，才自动启动所有的节点以求从网络分区中恢复。



​		在`pause-if-all-down`模式下，`RabbitMQ`集群中的节点在和所配置的列表中的任何节点不能交互时才会关闭。此模式下只有`ignore`和`autoheal`两种配置。

​		如果一个节点与另一个节点无法通信时，则会关闭自身的`RabbitMQ`应用。如果是某节点本身发生了故障造成网络不可用，而其他节点都是正常的情况下，这

种规则会让所有的节点中`RabbitMQ`应用都关闭，待该节点中的网络恢复之后，各个节点再启动自身应用以从网络分区中恢复。



​		在`autoheal`模式下，当认为发生网络分区时，`RabbitMQ`会自动决定一个获胜的分区，然后重启不在这个分区中的节点来从网络分区中恢复。一个获胜的分区

是指客户端连接最多的分区，如果产生一个平局，即有两个或者多个分区的客户端连接数一样多，那么节点数最多的一个分区就是获胜分区。如果此时节点数也一

样多，将以节点名称的字典序来挑选获胜分区。



​		`ignore`模式：发生网络分区时，不做任何动作，需要人工介入。

​		`pause-minority`模式：对于对等分区的处理不够优雅，可能会关闭所有的节点。一般情况下，可应用于非跨机架、奇数节点数的集群中。

​		`pause-if-all-down`模式：对于受信节点的选择尤为考究，尤其是在集群中所有节点硬件配置相同的情况下。此种模式可以处理对等分区的情形。

​		`autoheal`模式：可以处于各个情形下的网络分区。但是如果集群中有节点处于非运行状态，则此种模式会失效。





