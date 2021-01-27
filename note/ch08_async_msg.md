> * 格式形如`1.2.1`的章节序号为原书的章节序号
> * 格式形如`(1)/(2)/(3)`的章节序号为在笔记中展开的内容

# CH08 异步消息

> 本章以Taco订单创建之后、等待厨房烹饪完成为例子，分别演示基于三种不同消息队列的异步通信实现
>
> Demo项目：[../ch08/](../ch08/)

## 8.1 使用JMS

> JMS（Java Message Service）是2001年推出的Message Broker通用API规范，它的出现使Broker API得到统一

### 8.1.1 环境搭建

#### (1) 在项目中添加JMS客户端

##### (a) 底层使用Apache ActiveMQ （Legacy）

###### 依赖项

> ~~~xml
> <dependency>
>   <groupId>org.springframework.boot</groupId>
>   <artifactId>spring-boot-starter-activemq</artifactId>
> </dependency>
> ~~~

###### Spring配置

> | Property                   | Description                                                 |
> | -------------------------- | ----------------------------------------------------------- |
> | spring.activemq.broker-url | The URL of the broker                                       |
> | spring.activemq.user       | The user to use to access the broker (optional)             |
> | spring.activemq.password   | The password to use to access the broker (optional)         |
> | spring.activemq.in-memory  | Whether or not to start an in-memory broker (default: true) |
>
> ~~~yml
> spring:
>   activemq:
>     # 不设置代理主机和端口，而是使用url来指定代理的地址
>     broker-url: tcp://activemq.tacocloud.com
>     user: tacoweb
>     password: l3tm31n
> ~~~
>
> 另外需要将生产环境的`spring.activemq.in-memory`设置为false，否则会启动内存中运行的嵌入式代理

###### 安装和启动

> [http://activemq.apache.org/getting-started.html#GettingStarted-Pre-InstallationRequirements](http://activemq.apache.org/getting-started.html#GettingStarted-Pre-InstallationRequirements)

##### (b) 底层使用ActiveMQ Artemis

###### 依赖项 

> pom.xml：[/tacocloud-messaging-jms/pom.xml](../ch08/tacocloud-messaging-jms/pom.xml)
>
> ~~~xml
> <dependency>
>   <groupId>org.springframework.boot</groupId>
>   <artifactId>spring-boot-starter-artemis</artifactId>
> </dependency>
> ~~~
>
> ActiveMQ Artemis是重新实现的下一代ActiveMQ，因此这一章的demo项目基于ActiveMQ Artemis，但是在编写代码方面，两种选择没有差别

###### Spring配置

> | Property                | Description                                         |
> | ----------------------- | --------------------------------------------------- |
> | spring.artemis.host     | The broker’s host                                   |
> | spring.artemis.port     | The broker’s port                                   |
> | spring.artemis.user     | The user to use to access the broker (optional)     |
> | spring.artemis.password | The password to use to access the broker (optional) |
>
> ~~~yml
> ---
> spring:
>   profiles: production
>     
>   artemis:
>     host: artemis.tacocloud.com
>     port: 61617
>     user: tacoweb
>     password: l3tm31n
> ~~~
>
> 完整配置：[/tacocloud-messaging-jms/src/main/resources/application.yml](../ch08/tacocloud-messaging-jms/src/main/resources/application.yml)

###### 安装和启动

> [https://activemq.apache.org/artemis/docs/latest/using-server.html](https://activemq.apache.org/artemis/docs/latest/using-server.html)

### 8.1.2 使用JMS Template发送消息

> Demo项目地址：[/tacocloud-messaging-jms/](../ch08/tacocloud-messaging-jms/)

#### (1) 发送消息

##### (a) 发送消息的9个方法

> JMS starter依赖添加之后，Spring Boot会自动配置Jms Template，它封装了创建连接、会话、处理异常等大量的通用操作。以下是JMSTemplate用来发送消息的API
>
> ~~~java
> // 发送原始消息
> // * 需要MessageCreator来生成Message对象
> void send(/*发往默认目的地*/ MessageCreator messageCreator) throws JmsException;
> void send(/*发往指定目的地*/ Destination destination, MessageCreator messageCreator) throws JmsException;
> void send(/*发往指定目的地*/ String destinationName, MessageCreator messageCreator) throws JmsException;
> 
> // 发送根据对象转换而成的消息
> // * 自动将Object转换成Message
> void convertAndSend(/*发往默认目的地*/ Object message) throws JmsException;
> void convertAndSend(/*发往指定目的地*/ Destination destination, Object message) throws JmsException;
> void convertAndSend(/*发往指定目的地*/ String destinationName, Object message) throws JmsException;
> 
> // 发送根据对象转换而成的消息，并且带有后期处理功能
> // * 自动将Object转换成Message
> // * 可通过MessagePostProcessor来在发送前对Message进行自定义
> void convertAndSend(/*发往默认目的地*/ Object message,
>             MessagePostProcessor postProcessor) throws JmsException;
> void convertAndSend(/*发往指定目的地*/ Destination destination, Object message,
>             MessagePostProcessor postProcessor) throws JmsException;
> void convertAndSend(/*发往指定目的地*/ String destinationName, Object message,
>             MessagePostProcessor postProcessor) throws JmsException;
> ~~~

##### (b) `send(..., MessageCreator)` 

###### 发往默认目的地

> ```java
> @Service
> public class JmsOrderMessagingService implements OrderMessagingService {
> 	private JmsTemplate jms;
> 
> 	@Autowired
> 	public JmsOrderMessagingService(JmsTemplate jms) {
> 		this.jms = jms;
> 	}
> 
>     @Override
> 	public void sendOrder(Order order) {
>         // 发往默认目的地
> 		jms.send(session -> session.createObjectMessage(order));
>         // 用lambda表达式实现函数式接口MessageCreator，上面的代码相当于
>         // 这个接口的作用是把要发送的对象转换成Message
> 		// 	jms.send(new MessageCreator() {
> 		// 		@Override
> 		// 		public Message createMessage(Session session) throws JMSException {
> 		// 			return session.createObjectMessage(order);
> 		// 		}
> 		// 	});
>     }
> }
> ```
>
> 配置默认的目的地
>
> ~~~yml
> spring:
> 	jms:
> 		template:
> 			default-destination: tacocloud.order.queue
> ~~~
>
> 完整配置：[/tacocloud-messaging-jms/src/main/resources/application.yml](../ch08/tacocloud-messaging-jms/src/main/resources/application.yml)

###### 指定目的地

> 在某个Java Configuration类中创建一个Destination Bean
>
> ~~~java
> @Bean
> public Destination orderQueue() {
>     // 如果使用ActiveMQ Artemis，使用org.apache.actrivemq.artemis.jms.client.ActiveMQQueue
>     // 如果使用Apache ActiveMQ (Legacy)，使用org.apache.activemq.command.ActiveMQQueue
> 	return new ActiveMQQueue("tacocloud.order.queue");
> }
> ~~~
>
> 将这个Bean注入到service中
>
> ~~~java
> @Service
> public class JmsOrderMessagingService implements OrderMessagingService {
>     // 注入JmsTemplate和Destination
>     private JmsTemplate jms;
> 	private Destination orderQueue;
> 	@Autowired
> 	public JmsOrderMessagingService(JmsTemplate jms, Destination orderQueue) {
> 		this.jms = jms;
> 		this.orderQueue = orderQueue;
> 	}
> 
> 	...
> 
> 	@Override
> 	public void sendOrder(Order order) {
> 		jms.send(
>             orderQueue,  //消息发送目的地
>             session -> session.createObjectMessage(order) // MessageCreator
>         );
> 	}
> }
> ~~~

##### (c) `convertAndSend(..., Object msg)`

> 对`sendOrder`进行了简化，不再需要传入MessageCreator来把对象转换成Message，这由框架来执行
>
> 同样拥有三种重载方式（1）发往默认destination（2）用String指定destination name（3）用Destination对象来指定发送目标 。下面只列出一种
>
> ~~~java
> @Override
> public void sendOrder(Order order) {
> 	jms.convertAndSend("tacocloud.order.queue", order);
> }
> ~~~

#### (2) 配置`MessageConverter`

> 不论选择9个方法中的哪一个，都需要把要发送的对象转换成Message（通过自定义MessageCreator或者交给框架来执行），而这个操作都是通过`MessageConverter`来完成。

##### (a) `MesageConverter`接口

> ~~~java
> public interface MessageConverter {
> 	Message toMessage(Object object, Session session) throws JMSException, MessageConversionException;
> 	Object fromMessage(Message message)
> }
> ~~~

##### (b) Spring内置的`MessageConverter`实现类

> | Message converter               | What it does                                                 |
> | ------------------------------- | ------------------------------------------------------------ |
> | MappingJackson2MessageConverter | Uses the Jackson 2 JSON library to convert messages to and from `JSON` |
> | MarshallingMessageConverter     | Uses JAXB to convert messages to and from `XML`              |
> | MessagingMessageConverter       | 使用（1）底层的MessageConverter实现消息抽象的Message载荷与javax.jms.Message之间的转换（2）JmsHeaderMapper实现JMS头信息与标准消息头信息之间的转换 |
> | SimpleMessageConverter          | 实现（1）String与TextMessage（2）字节数组与BytesMessage（3）Map与MapMessage（3）Serializable对象与ObjectMessage之间的相互转换 |

##### (c) 指定`MessageConverer`的具体实现

> 默认的转换器是`SimpleMessageConverter`，但是它需要被发送的对象实现了Serializable接口。因此很多时候，会更改默认实现，例如使用MappingJackson2MessageConverter等更为方便的转换器。
>
> 例子：[/tacocloud-messaging-jms/src/main/java/tacos/messaging/MessagingConfig.java](../ch08/tacocloud-messaging-jms/src/main/java/tacos/messaging/MessagingConfig.java)
>
> ~~~java
> @Bean
> public MappingJackson2MessageConverter messageConverter() {
>    	// 将要发送的对象与Json进行转换
> 	MappingJackson2MessageConverter messageConverter = new MappingJackson2MessageConverter();
> 	// JSON中的"_typeId"字段用来上接收者知道传入的消息要转换成什么类型
>    	messageConverter.setTypeIdPropertyName("_typeId");
> 	// 在发送端、发送时带有"order"类型ID的JSON与Order类相对应
>    	// 在接收端、也要配置类似的消息转换器，将”order“类型ID的JSON映射成它能够理解的某个类
> 	Map<String, Class<?>> typeIdMappings = new HashMap<String, Class<?>>();
> 	typeIdMappings.put("order", Order.class);
> 	messageConverter.setTypeIdMappings(typeIdMappings);
> 
> 	return messageConverter;
> }
> ~~~

#### (3) Message Post Processing

> JMSTemplate的三个方法，可以在Object转换成Message之后，执行`后期处理操作`
>
> ~~~java
> // 发送根据对象转换而成的消息，并且带有后期处理功能
> // * 自动将Object转换成Message
> // * 可通过MessagePostProcessor来在发送前对Message进行自定义
> void convertAndSend(/*发往默认目的地*/ Object message,
>             MessagePostProcessor postProcessor) throws JmsException;
> void convertAndSend(/*发往指定目的地*/ Destination destination, Object message,
>             MessagePostProcessor postProcessor) throws JmsException;
> void convertAndSend(/*发往指定目的地*/ String destinationName, Object message,
>             MessagePostProcessor postProcessor) throws JmsException;
> ~~~
>
> 一个例子是，将订单（Order）对象转换成Message之后，在消息头中加上添加自定义字段`"X_ORDER_SOURCE", "WEB"`来表示该订单是网络订单而非实体店订单
>
> ~~~java
> @Override
> public void sendOrder(Order order) {
> 	jms.convertAndSend(
>         "tacocloud.order.queue", // 发送目的地
>         order,					 // 被发送的对虾干
>         this::addOrderSource	 // 用lambda表达式实现的函数式接口MessagePostProcessor
>     );
> }
> 
> // lambda表达式所执行的方法，在消息头中加上"X_ORDRE_SOURCE"字段
> private Message addOrderSource(Message message) throws JMSException {
> 	message.setStringProperty("X_ORDER_SOURCE", "WEB");
> 	return message;
> }
> ~~~
>
> 代码：[/tacocloud-messaging-jms/src/main/java/tacos/messaging/JmsOrderMessagingService.java](../ch08/tacocloud-messaging-jms/src/main/java/tacos/messaging/JmsOrderMessagingService.java)

### 8.1.3 接收JMS消息

#### (1) 使用`JmsTemplate`从broker拉取消息（pull message）

> `拉取消息`会产生阻塞，但是相比`监听器`的 好处时，当消息抵达过快时，不容易产生过载
>
> 项目地址：[../ch08/tacocloud-kitchen/](../ch08/tacocloud-kitchen/)

##### (a) 用于拉取消息的方法

>  `JmsTemplate`用于拉取消息的6个方法如下
>
> ~~~java
> // 接收原始消息
> Message receive() throws JmsException;
> Message receive(Destination destination) throws JmsException;
> Message receive(String destinationName) throws JmsException;
> 
> // 框架会执行转换操作，将原始消息转换成对相应Object
> Object receiveAndConvert() throws JmsException;
> Object receiveAndConvert(Destination destination) throws JmsException;
> Object receiveAndConvert(String destinationName) throws JmsException;
> ~~~

##### (b) `receive(...)`

> receive返回原始的Message对象，需要自己调用MessageConvertor来提取message payload
>
> ~~~java
> @Component
> public class JmsOrderReceiver implements OrderReceiver {
> 	// 注入JmsTemplate和MessageConverter
>    	private JmsTemplate jms;
> 	private MessageConverter converter;
> 	@Autowired
> 	public JmsOrderReceiver(JmsTemplate jms, MessageConverter converter) {
> 		this.jms = jms;
> 		this.converter = converter;
> 	}
> 
>    	// 从broker获取原始Message，并调用converter将其转换为Order对象
> 	public Order receiveOrder() {
> 		Message message = jms.receive("tacocloud.order.queue" /*destination name*/);
> 		return (Order) converter.fromMessage(message);
> 	}
> }
> ~~~

##### (b) `receiveAndConvert(...)`

> 不需要message header之类的数据，只需要message payload时，可以调用receiveAndConvert方法由框架来调用MessageConvertor进行payload提取
>
> ~~~java
> // 只在spring.profiles配置为"jms-listener"才会创建这个bean
> // 这是因为该demo项目实现了多种消息接收方式，需要通过profile指定启动哪一种
> @Profile("jms-template")
> @Component
> public class JmsOrderReceiver implements OrderReceiver {
> 	// 只注入JmsTemplate，不需要注入MessageConvertor
> 	private JmsTemplate jms;
> 	@Autowired
> 	public JmsOrderReceiver(JmsTemplate jms) {
> 		this.jms = jms;
> 	}
> 
> 	// 框架会使用MessageConverter从Message中提取payload并转换成Order对象
> 	public Order receiveOrder() {
> 		return (Order) jms.receiveAndConvert("tacocloud.order.queue");
> 	}
> }
> ~~~
>
> 完整代码：[/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/jms/JmsOrderReceiver.java](../ch08/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/jms/JmsOrderReceiver.java)

#### (2) 使用`@JmsListener`来让JMS推送消息给消费者

> 消息监听器的有点是，不会阻塞，并且在消息到达时快速处理
>
> 但如果消息处理的瓶颈在于下游时，使用消息监听器未必是好的选择，当消息处理器需要根据自己状况来决定是否接受更多信息时，拉取可能是更好的模式
>
> 例如这个例子，处理的瓶颈在于下游的厨师，监听器带来的快速处理并不能加快taco的制作反而会给厨房人员带来过重的负担

##### (a) 创建消息监听器

> ~~~java
> // 只在spring.profiles配置为"jms-listener"才会创建这个bean
> // 这是因为该demo项目实现了多种消息接收方式，需要通过profile指定启动哪一种
> @Profile("jms-listener") 
> @Component
> public class OrderListener {
>    	// 注入用来处理消息的类，它的方法会在接收到消息后背调用
> 	private KitchenUI ui;
> 	@Autowired
> 	public OrderListener(KitchenUI ui) {
> 		this.ui = ui;
> 	}
>     
>    	// 创建JmsListener，该方法会在收到消息后被调用
> 	@JmsListener(destination = "tacocloud.order.queue")
> 	public void receiveOrder(Order order) {
> 		ui.displayOrder(order);
> 	}
> }
> ~~~
>
> 完整代码：[/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/jms/listener/OrderListener.java](../ch08/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/jms/listener/OrderListener.java)

#### (3) 定义MessageConvertor

> 上面(1)(2)的例子中，都直接或借助框架间接使用了`MessageConverter` 。完整的使用方法参考`8.1.2.(2)`小节，下面的代码是该demo中使用的MessageConvertor
>
> 完整代码：[/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/jms/MessagingConfig.java](../ch08/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/jms/MessagingConfig.java)
>
> ```java
> // 只在spring.profiles配置为"jms-template"，"jms-listener"才会创建Bean
> // 为(1)(2)的两种消息接收方式提供支持
> @Profile({"jms-template", "jms-listener"})
> @Configuration
> public class MessagingConfig {
> 	@Bean
> 	public MappingJackson2MessageConverter messageConverter() {
> 		// 将要发送的对象与JSOn进行转换
> 		MappingJackson2MessageConverter messageConverter = new MappingJackson2MessageConverter();
> 		// JSON中的"_typeId"字段用来上接收者知道传入的消息要转换成什么类型        
> 		messageConverter.setTypeIdPropertyName("_typeId");
> 	
>         // 将typeid为”order“的JSON映射成它能够理解的某个类
> 		Map<String, Class<?>> typeIdMappings = new HashMap<String, Class<?>>();
> 		typeIdMappings.put("order", Order.class);
> 		messageConverter.setTypeIdMappings(typeIdMappings);
> 
> 		return messageConverter;
>    }
> }
> ```

#### (4) JMS的缺点

>  作为Java的消息规范、它只能用在Java应用中，不能用在JVM以外的其他语言和平台

## 8.2 使用RabitMQ和AMQP

### (1) RabbitMQ的优势及消息路由过程 

`RabbitMQ`是`AMQP`（`Advanced Message Queueing Protocal`）的一种实现，相比`JMS` 有以下优势

> *  `Rabbit`与`Kafka`都支持JVM以外语言和平台的消息传递
> * `Rabbit`使用`exchange`和`routing key`来寻址，能够将消息队列与消费者解耦

消息传递过程如下

> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/spring_in_action_5_rabbitmq_exchange.jpg)
>
> (1) 消息到达broker（2）Exchange将消息路由到1个或多个队列中（根据exchange的类型、exchange和队列之前的binding、消息routing key）

###  (2) 六种Exchange

> * `Default Exchange`：将消息路由至`名字与消息routingkey相同的队列`（所有的队列都会自动绑定至Default Exchange）。
> * `Fanout Exchange`：消息路由到`所有`绑定队列上。
> * `Direct Exchange`：如果`消息的routingkey`与`队列的bindingkey`相同，那么消息将会路由到该队列上。
> * `Topic Exchange`：如果`消息的routingkey`与`队列bindingkey（可能会包含通配符）匹配`，那么消息将会路由到一个或多个这样的队列上。
> * `Headers Exchange`：与TopicExchange类似，只不过要`基于消息的头信息`（而不是routingkey）进行路由。
> * `Deadletter Exchange`：捕获所有无法投递（也就是它们无法匹配所有已定义的Exchange和队列的binding关系）的消息。

###  (3) Demo项目及参考资料

> Demo项目
>
> * 生产者：[../ch08/tacocloud-messaging-rabbitmq/](ch08/tacocloud-messaging-rabbitmq/)
> * 消费者：[../ch08/tacocloud-kitchen/](../ch08/tacocloud-kitchen/)
>
> 关于Rabbit MQ的更多内容，参考《RabbitMQ in  Action》

### 8.2.1 添加RabbitMQ和AMQP

#### (1) 添加依赖

> 添加AMQP starter依赖，会触发Spring Boot自动配置，创建AMQP连接工厂以及RabbitTemplate bean以及其他支持组件
>
> ~~~xml
> <dependency>
> 	<groupId>org.springframework.boot</groupId>
> 	<artifactId>spring-boot-starter-amqp</artifactId>
> </dependency>
> ~~~
>
> 生产者：[/tacocloud-messaging-rabbitmq/pom.xml](../ch08/tacocloud-messaging-rabbitmq/pom.xml)
>
> 消费者：[/tacocloud-kitchen/pom.xml](../ch08/tacocloud-kitchen/pom.xml)

#### (2)  配置

> 配置项
>
> | Property                  | Description                                         |
> | ------------------------- | --------------------------------------------------- |
> | spring.rabbitmq.addresses | A comma-separated list of RabbitMQ broker addresses |
> | spring.rabbitmq.host      | The broker’s host (defaults to localhost)           |
> | spring.rabbitmq.port      | The broker’s port (defaults to 5672)                |
> | spring.rabbitmq.username  | The username to use to access the broker (optional) |
> | spring.rabbitmq.password  | The password to use to access the broker (optional) |
>
> 配置文件例子
>
> ~~~yml
> spring:
> 	profiles: prod
> 	rabbitmq:
> 		host: rabbit.tacocloud.com
> 		port: 5673
> 		username: tacoweb
> 		password: l3tm31n
> ~~~

### 8.2.2 使用RabbitTemplate发送消息

#### (1) 用于发送消息的方法

RabbitTemplate接口中常用的三类方法

> ~~~java
> // 发送原始消息，需要自己写代码调用MessageConvertor把要发送的数据转换成Message
> void send(Message message) throws AmqpException;
> void send(String routingKey, Message message) throws AmqpException;
> void send(String exchange, String routingKey, Message message) throws AmqpException;
> 
> // 框架自动调用MessageConvertor将要发送的对象转换成Message
> void convertAndSend(Object message) throws AmqpException;
> void convertAndSend(String routingKey, Object message) throws AmqpException;
> void convertAndSend(String exchange, String routingKey, Object message) throws AmqpException;
> 
> // 框架自动调用MessageConvertor将要发送的对象转换成Message
> // 同时还可以传入MessagePostProcess、在转换完成后对Message做后期处理（例如向Message Header中添加内容等）
> void convertAndSend(Object message, MessagePostProcessor mPP) throws AmqpException;
> void convertAndSend(String routingKey, Object message, MessagePostProcessor messagePostProcessor) throws AmqpException;
> void convertAndSend(String exchange, String routingKey, Object message, MessagePostProcessor messagePostProcessor) throws AmqpException;
> ~~~

每一类方法都有3个参数重载形式

> 方式1：发往默认的exchange和默认的routing key
>
> 方式2：发往默认的exchange和指定的routing key
>
> 方式3：发往指定的exchange和指定的routing key

#### (2) `默认的exchange`和`默认routing key`

> 在不做配置的情况下
>
> * 默认exchange的名字是“”（空String）、对应RabbitMQ broker自动生成的`Deafult Exchange `
> * 默认rouging. key的名字是“”（空String）、它的路由取决于Exchange及响应的binding
>
> 可以通过修改配置来改变这两个值
>
> ~~~yml
> spring:
> 	rabbitmq:
> 		template:
> 			exchange: tacocloud.orders
> 			routing-key: kitchens.central
> ~~~

#### (3) 三类发送方法的使用及例子

##### (a) `send(...)`

> ~~~java
> @Service
> public class RabbitOrderMessagingService implements OrderMessagingService {
> 	// 注入RabbitTemplate
>     private RabbitTemplate rabbit;
> 	@Autowired
> 	public RabbitOrderMessagingService(RabbitTemplate rabbit) {
> 		this.rabbit = rabbit;
> 	}
> 
> 	public void sendOrder(Order order) {
>         // 调用MessageConverter将Order对象转换成Message
> 		MessageConverter converter = rabbit.getMessageConverter();
> 		MessageProperties props = new MessageProperties();
> 		Message message = converter.toMessage(order, props);
>         // 发送到名为"tacocloud.order"的exchange的默认routing key上
>         // 根据上面的配置，这个默认routing key是"kitchens.central"
> 		rabbit.send("tacocloud.order", message);
> 	}
> }
> ~~~

##### (b) `convertAndSend(...)`及`MessagePostProcessor`

> 使用convertAndSend，可以省去调用MessageConverter的步骤
>
> ~~~java
> public void sendOrder(Order order) {
>     // 不需要手动调用MessageConverter，框架会自动调用
> 	rabbit.convertAndSend("tacocloud.order", order);
> }
> ~~~
>
> 另外还可以指定一个`MessagePostProcessor`，在Order对象转换为Message之后，对Message做后期处理
>
> ~~~java
> @Service
> public class RabbitOrderMessagingService implements OrderMessagingService {
> 	// 注入RabbitTemplate
> 	private RabbitTemplate rabbit;
> 	@Autowired
> 	public RabbitOrderMessagingService(RabbitTemplate rabbit) {
> 		this.rabbit = rabbit;
> 	}
> 	
> 	public void sendOrder(Order order) {
> 		rabbit.convertAndSend(
>             "tacocloud.order.queue", order,
>             // 后期操作，在Message Header中添加订单来源，表示该订单是网络订单而非实体店订单
> 			new MessagePostProcessor() {
>                 @Override
>                 public Message postProcessMessage(Message message) throws AmqpException {
>                     MessageProperties props = message.getMessageProperties();
>                     props.setHeader("X_ORDER_SOURCE", "WEB");
>                     return message;
>                 } 
> 		});
> 	}
> }
> ~~~
>
> 完整代码：[/tacocloud-messaging-rabbitmq/src/main/java/tacos/messaging/RabbitOrderMessagingService.java](../ch08/tacocloud-messaging-rabbitmq/src/main/java/tacos/messaging/RabbitOrderMessagingService.java)

#### (4) `MessageConverter`

##### (a) Spring为RabbitTemplate提供的MessageConverter

> * `Jackson2JsonMessageConverter`：使用Jackson2JSON实现对象和JSON的相互转换。
> * `MarshallingMessageConverter`：使用Spring的Marshaller和Unmarshaller进行转换。
> * `SerializerMessageConverter`：使用Spring的Serializer和Deserializer转换String和任意种类的原生对象。
> * `SimpleMessageConverter`：转换String、字节数组和Serializable类型
> * `ContentTypeDelegatingMessageConverter`：基于contentType头信息，将转换功能委托给另外一个MessageConverter。
> * `MessagingMessageConverter`：
>     * 将消息转换功能委托给另外一个MessageConverter
>     * 将头信息的转换委托给AmqpHeaderConverter。

##### (b) 设置MessageConverter

> 默认值为`SimpleMessageConverter`，可以使用如下的方法设置其他的Converter，Spring会自动将其装配到RabbitTemplate中 
>
> ~~~java
> @Bean
> public Jackson2JsonMessageConverter messageConverter() {
> 	return new Jackson2JsonMessageConverter();
> }
> ~~~
>
> 完整代码：[/tacocloud-messaging-rabbitmq/src/main/java/tacos/messaging/MessagingConfig.java](../ch08/tacocloud-messaging-rabbitmq/src/main/java/tacos/messaging/MessagingConfig.java)

### 8.2.3 接收来自RabbitMQ的消息

> 同样有两种方法（1）使用RabbitTemplate拉取消息（2）让broker把消息推送到带有@RabbitListener注解的方法

#### (1) 使用RabbitTemplate拉取消息

##### (a) 用于拉取消息的方法

RabbitTemplate接口中用于拉取消息的方法

> ~~~java
> // 介绍原始的Message
> Message receive() throws AmqpException;
> Message receive(String queueName) throws AmqpException;
> Message receive(long timeoutMillis) throws AmqpException;
> Message receive(String queueName, long timeoutMillis) throws AmqpException;
> 
> // 接收由消息转换而成的对象（框架自动调用MessageConveter）
> Object receiveAndConvert() throws AmqpException;
> Object receiveAndConvert(String queueName) throws AmqpException;
> Object receiveAndConvert(long timeoutMillis) throws AmqpException;
> Object receiveAndConvert(String queueName, long timeoutMillis) throws AmqpException;
> 
> //接收由消息转换而成的类型安全的对象（框架自动调用MessageConveter）
> <T> T receiveAndConvert(ParameterizedTypeReference<T> type) throws AmqpException;
> <T> T receiveAndConvert(String queueName, ParameterizedTypeReference<T> type) throws AmqpException;
> <T> T receiveAndConvert(long timeoutMillis, ParameterizedTypeReference<T> type) throws AmqpException;
> <T> T receiveAndConvert(String queueName, long timeoutMillis, ParameterizedTypeReference<T> type) throws AmqpException;
> ~~~

说明：

> queueName：
>
> * 对于生产者来说，需要指定exchange和routing key，随后broker根据这两个值将消息mapping到对应的队列上
>
> * 对于消费者来说，它只需要指定从哪个队列接收消息，因此方法参数中只需要queryName（而不需要exchange和routing key）
>
> timeoutMillis：设置为0时，方法不会阻塞，没有消息时会返回null

##### (b) `receive(...)`

> ~~~java
> public Order receiveOrder() {
> 	// 从队列"tacocloud.order.queue"接收消息，超时时仍然没有收到消息，则返回null
> 	Message message = rabbit.receive("tacocloud.order.queue", 30000);
> 	// 如果配置了spring.rabbitmq.template.receive-timeout则不需要传入超时时间，直接调用receive(queueName)即可
> 	// Message message = rabbit.receive("tacocloud.order.queue");        
> 	return message != null
> 			? (Order) converter.fromMessage(message) // 手动调用MessageConveter
> 			: null;
> }
> ~~~

##### (c) `receiveAndConvert(...)`

> ~~~java
> // 只在spring.profiles配置为"rabbitmq-template"才会创建这个bean
> // 因为Demo项目为多种消息队列提供了实现，使用spring.profiles来决定装配哪个bean
> @Profile("rabbitmq-template") 
> @Component("templateOrderReceiver")
> public class RabbitOrderReceiver implements OrderReceiver {
> 	// 注入RabbitTemplate
> 	private RabbitTemplate rabbit;
> 	public RabbitOrderReceiver(RabbitTemplate rabbit) {
> 		this.rabbit = rabbit;
> 	}
>  
> 	public Order receiveOrder() {
> 		// 由框架来调用MessageConverter将Message转成Object
> 		return (Order) rabbit.receiveAndConvert("tacocloud.order.queue");
> 		// 另一种方式，使用ParameterizedTypeReference<Order>
>         // * 不仅由框架来调用MessageConverter，也有框架来进行类型检查和转型（Object → Order）
>         // * 但是需要MessageConverter实现SmartMessageConverter接口
>         // * 在Spring内置的候选中，Jackson2JsonMessageConverter是唯一的选择
> 		// return rabbit.receiveAndConvert(
> 		//    "tacocloud.order.queue", new ParameterizedTypeReference<Order>() {});      
> 	}
> }
> ~~~
>
> 完整代码：[/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/rabbit/RabbitOrderReceiver.java](../ch08/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/rabbit/RabbitOrderReceiver.java)

#### (2) 使用`@RabbitListener`处理broker推送的消息

> ~~~java
> // 只在spring.profiles配置为"rabbitmq-listener"才会创建这个bean
> // 因为Demo项目为多种消息队列提供了实现，使用spring.profiles来决定装配哪个bean
> @Profile("rabbitmq-listener")
> @Component
> public class OrderListener {
> 	// 注入用来处理消息的类，它的方法会在接收到消息后背调用
> 	private KitchenUI ui;
> 	@Autowired
> 	public OrderListener(KitchenUI ui) {
> 		this.ui = ui;
> 	}
> 
>     // 创建RabbitListner，该方法在接收到消息后被调用
> 	@RabbitListener(queues = "tacocloud.order.queue")
> 	public void receiveOrder(Order order) {
> 		ui.displayOrder(order);
> 	} 
> }
> ~~~
>
> 完整代码：[/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/rabbit/listener/OrderListener.java](../ch08/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/rabbit/listener/OrderListener.java)

#### (3) 定义MessageConvertor

> 上面(1)(2)的例子中，都直接或借助框架间接使用了`MessageConverter` 。完整的使用方法参考`8.2.2.(4)`小节，下面的代码是该demo中使用的MessageConvertor
>
> 完整代码：[/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/rabbit/MessagingConfig.java](../ch08/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/rabbit/MessagingConfig.java)
>
> ~~~java
> @Profile({"rabbitmq-template", "rabbitmq-listener"})
> @Configuration
> public class MessagingConfig {
> 	@Bean
> 	public Jackson2JsonMessageConverter messageConverter() {
> 		return new Jackson2JsonMessageConverter();
> 	}
> }
> ~~~

## 8.3 使用Kafka

> Kafka设计为集群运行，扩展性强。它通过topic来进行消息的订阅和发布。topic下的数据会在集群节点上进行分区和复制
>
> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/spring_in_action_6_kafka_cluster.jpg)
>
> Demo项目：
>
> * 生产者：[/ch08/tacocloud-messaging-kafka/](../ch08/tacocloud-messaging-kafka/pom.xml)
> * 消费者：[/ch08/tacocloud-kitchen/](../ch08/tacocloud-kitchen/pom.xml)

### 8.3.1 在项目中引入Spring Kafka

#### (1) 依赖

> 引入该依赖后，Spring Boot会自动配置Kafka并生成KafkaTemplate
>
> ~~~xml
> <dependency>
>   <groupId>org.springframework.kafka</groupId>
>   <artifactId>spring-kafka</artifactId>
> </dependency>
> ~~~
>
> * 生产者：[](../ch08/tacocloud-messaging-kafka/pom.xml)
> * 消费者：[](../ch08/tacocloud-kitchen/pom.xml)

#### (2) 配置

> 生产者配置例子：[/tacocloud-messaging-kafka/src/main/resources/application.yml](../ch08/tacocloud-messaging-kafka/src/main/resources/application.yml)
>
> ```yml
> # The values given here are actually the default values. But they are explicitly
> # set here as an example of setting the Kafka properties.
> spring:
> 	kafka:
> 		bootstrap-servers:
> 		- localhost:9092
> 		template:
> 			default-topic: tacocloud.orders.topic
> 		producer:
> 			keySerializer: org.springframework.kafka.support.serializer.JsonSerializer
> 			valueSerializer: org.springframework.kafka.support.serializer.JsonSerializer
> ```
>
> 消费者配置例子：[/tacocloud-kitchen/src/main/resources/application.yml](../ch08/tacocloud-kitchen/src/main/resources/application.yml)
>
> ```yml
> spring:
> 	kafka:
> 		bootstrap-servers:
> 		- localhost:9092
> 		template:
> 			default-topic: tacocloud.orders.topic
> 		consumer:
> 			value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
> 			group-id: tacocloud_kitchen
> 			properties:
> 				spring.json.trusted.packages: tacos
> ```
>
> 通常在生产环境下bootstrap server至少有三台
>
> ~~~yml
> spring:
>   kafka:
>     bootstrap-servers:
>     - kafka1.tacocloud.com:9092
>     - kafka2.tacocloud.com:9092
>     - kafka3.tacocloud.com:9092
> ~~~

### 8.3.2 使用KafkaTemplate发送消息

#### (1) 用于发消息的方法

> KafkaTemplate用于发消息的方法如下
>
> ~~~java
> ListenableFuture<SendResult<K, V>> send(String topic, V data);
> ListenableFuture<SendResult<K, V>> send(String topic, K key, V data);
> ListenableFuture<SendResult<K, V>> send(String topic, Integer partition, K key, V data);
> ListenableFuture<SendResult<K, V>> send(String topic, Integer partition, Long timestamp, K key, V data);
> ListenableFuture<SendResult<K, V>> send(ProducerRecord<K, V> record);
> ListenableFuture<SendResult<K, V>> send(Message<?> message);
> ListenableFuture<SendResult<K, V>> sendDefault(V data);
> ListenableFuture<SendResult<K, V>> sendDefault(K key, V data);
> ListenableFuture<SendResult<K, V>> sendDefault(Integer partition, K key, V data);
> ListenableFuture<SendResult<K, V>> sendDefault(Integer partition, Long timestamp, K key, V data);
> ~~~
>
> Kafka的序列化、反序列化是通过`key-serializer/value-serializer/key-deserilizer/value-serializer`配置来实现的。因此不需要MessageConverter和`convertAndSend()`这样的方法
>
> 可以通过参数指定：
>
> * 消息要发送到的主题（send()方法的必选参数）
> * 主题要写入的分区（可选）
> * 记录上要发送的key（可选）
> * 时间戳（可选，默认为`System.currentTimeMillis()`）
> * Payload（必选）

#### (2) 例子和说明

> ```java
> @Service
> public class KafkaOrderMessagingService{
>     // 注入KafkaTemplate
> 	private KafkaTemplate<String, Order> kafkaTemplate;
> 	@Autowired
> 	public KafkaOrderMessagingService(KafkaTemplate<String, Order> kafkaTemplate) {
> 		this.kafkaTemplate = kafkaTemplate;
> 	}
> 
> 	@Override
> 	public void sendOrder(Order order) {
> 		kafkaTemplate.send("tacocloud.orders.topic", order);
>         // 因为在上面8.3.1.(2)小节已经配置了spring.kafka.template.default-topic为tacocloud.orders.topic
>         // 也可以直接使用下面的代码
>         // kafkaTemplate.sendDefault(order);
> 	}
> }
> ```
>
> 完整代码：[/tacocloud-messaging-kafka/src/main/java/tacos/messaging/KafkaOrderMessagingService.java](../ch08/tacocloud-messaging-kafka/src/main/java/tacos/messaging/KafkaOrderMessagingService.java)

###  8.3.3 编写Kafka监听器

> KafkaTemplate没有接收消息的方法，接收消息唯一的方法是编写Kafka监听器
>
> 方法1：Listener拿到消息载荷`ComsumerRecord<KeyType, ValueType>`
>
> ~~~java
> @Component
> @Slf4j
> public class OrderListener {
>     private KitchenUI ui;
> 	@Autowired
> 	public OrderListener(KitchenUI ui) {
> 		this.ui = ui;
> 	}
> 
> 	@KafkaListener(topics="tacocloud.orders.topic")
> 	public void handle(Order order, ConsumerRecord<String, Order> record) {
> 		log.info("Received from partition {} with timestamp {}", 
>                  record.partition(), record.timestamp());
> 		ui.displayOrder(order);
> 	}
> }
> ~~~
>
> 方法2：Listener拿到原始消息`Message<Valuetype>`
>
> ```java
> @Component
> @Slf4j
> public class OrderListener {
> 	private KitchenUI ui;
> 	@Autowired
> 	public OrderListener(KitchenUI ui) {
> 		this.ui = ui;
> 	}
> 
> 	@KafkaListener(topics="tacocloud.orders.topic")
> 	public void handle(Order order, Message<Order> message) {
> 		MessageHeaders headers = message.getHeaders();
> 		log.info("Received from partition {} with timestamp {}",
> 			headers.get(KafkaHeaders.RECEIVED_PARTITION_ID),
> 			headers.get(KafkaHeaders.RECEIVED_TIMESTAMP));
> 		ui.displayOrder(order);
> 	}
> }
> ```
>
> 完整代码：[/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/kafka/listener/OrderListener.java](../ch08/tacocloud-kitchen/src/main/java/tacos/kitchen/messaging/kafka/listener/OrderListener.java)

## 8.4 小结

> 略

