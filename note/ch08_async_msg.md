[TOC]

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

#### (1) 发送消息的9个方法

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

#### (2) `send(...)` 

##### (a) 发往默认目的地

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
>         // 用lambda表达式实现函数式接口，上面的代码相当于
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

##### (b) 指定目的地

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

#### (3) `convertAndSend(...)`

> 





代码：[/tacocloud-messaging-jms/src/main/java/tacos/messaging/JmsOrderMessagingService.java](../ch08/tacocloud-messaging-jms/src/main/java/tacos/messaging/JmsOrderMessagingService.java)

### 8.1.3 接收JMS消息

## 8.2 使用RabitMQ和AMQP

> AMQP（Advanced Message Queueing Protocal）

### 8.2.1 添加RabbitMQ和AMQP

### 8.2.2 使用RabbitTemplate发送消息

### 8.2.3 接收来自RabbitMQ的消息

## 8.3 使用Kafka

### 8.3.1 环境搭建

### 8.3.2 使用KafkaTemplate发送消息

###  8.3.3 编写Kafka监听器

