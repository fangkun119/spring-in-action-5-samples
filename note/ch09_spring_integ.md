> <!-- START doctoc generated TOC please keep comment here to allow auto update -->
> <!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
> <!--**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*-->

- [CH09 Spring Integration](#ch09-spring-integration)
  - [9.1 简单Integration Flow](#91-%E7%AE%80%E5%8D%95integration-flow)
    - [(1) 应用程序部分](#1-%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E9%83%A8%E5%88%86)
      - [(a) 添加依赖](#a-%E6%B7%BB%E5%8A%A0%E4%BE%9D%E8%B5%96)
      - [(b) 定义消息网关](#b-%E5%AE%9A%E4%B9%89%E6%B6%88%E6%81%AF%E7%BD%91%E5%85%B3)
      - [(c) 将数据发送到消息网关](#c-%E5%B0%86%E6%95%B0%E6%8D%AE%E5%8F%91%E9%80%81%E5%88%B0%E6%B6%88%E6%81%AF%E7%BD%91%E5%85%B3)
    - [(2) Integration Flow部分](#2-integration-flow%E9%83%A8%E5%88%86)
    - [9.1.1 使用XML配置](#911-%E4%BD%BF%E7%94%A8xml%E9%85%8D%E7%BD%AE)
      - [(1) 配置内容及说明](#1-%E9%85%8D%E7%BD%AE%E5%86%85%E5%AE%B9%E5%8F%8A%E8%AF%B4%E6%98%8E)
      - [(2) 将配置导入Spring应用](#2-%E5%B0%86%E9%85%8D%E7%BD%AE%E5%AF%BC%E5%85%A5spring%E5%BA%94%E7%94%A8)
    - [9.1.2 使用Java配置](#912-%E4%BD%BF%E7%94%A8java%E9%85%8D%E7%BD%AE)
      - [(1) 配置内容及说明](#1-%E9%85%8D%E7%BD%AE%E5%86%85%E5%AE%B9%E5%8F%8A%E8%AF%B4%E6%98%8E-1)
    - [9.1.3 使用DSL配置](#913-%E4%BD%BF%E7%94%A8dsl%E9%85%8D%E7%BD%AE)
      - [(1) 配置内容及说明](#1-%E9%85%8D%E7%BD%AE%E5%86%85%E5%AE%B9%E5%8F%8A%E8%AF%B4%E6%98%8E-2)
  - [9.2 Spring Integration功能概览](#92-spring-integration%E5%8A%9F%E8%83%BD%E6%A6%82%E8%A7%88)
    - [(1) Integration Flow中各个组件的功能](#1-integration-flow%E4%B8%AD%E5%90%84%E4%B8%AA%E7%BB%84%E4%BB%B6%E7%9A%84%E5%8A%9F%E8%83%BD)
    - [9.2.1 Message Channel](#921-message-channel)
      - [(1) Spring Integration提供的通道实现](#1-spring-integration%E6%8F%90%E4%BE%9B%E7%9A%84%E9%80%9A%E9%81%93%E5%AE%9E%E7%8E%B0)
      - [(2) 创建和使用](#2-%E5%88%9B%E5%BB%BA%E5%92%8C%E4%BD%BF%E7%94%A8)
        - [(a) 创建](#a-%E5%88%9B%E5%BB%BA)
        - [(b) 使用](#b-%E4%BD%BF%E7%94%A8)
        - [(c) QueueChannel](#c-queuechannel)
    - [9.2.2 Filter](#922-filter)
      - [(1) 功能](#1-%E5%8A%9F%E8%83%BD)
      - [(2) 例子](#2-%E4%BE%8B%E5%AD%90)
    - [9.2.3 Transformer](#923-transformer)
      - [(1) 功能](#1-%E5%8A%9F%E8%83%BD-1)
      - [(2) 例子](#2-%E4%BE%8B%E5%AD%90-1)
    - [9.2.4 Router](#924-router)
      - [(1) 功能](#1-%E5%8A%9F%E8%83%BD-2)
      - [(2) 例子](#2-%E4%BE%8B%E5%AD%90-2)
    - [9.2.5 Splitter](#925-splitter)
      - [(1) 功能](#1-%E5%8A%9F%E8%83%BD-3)
      - [(2) 典型应用场景](#2-%E5%85%B8%E5%9E%8B%E5%BA%94%E7%94%A8%E5%9C%BA%E6%99%AF)
      - [(3) 例子](#3-%E4%BE%8B%E5%AD%90)
    - [9.2.6 Service Activators](#926-service-activators)
      - [(1) 功能](#1-%E5%8A%9F%E8%83%BD-4)
      - [(2) 例子](#2-%E4%BE%8B%E5%AD%90-3)
    - [9.2.7 Gateway](#927-gateway)
      - [(1) 功能](#1-%E5%8A%9F%E8%83%BD-5)
      - [(2) 例子](#2-%E4%BE%8B%E5%AD%90-4)
    - [9.2.8 Channel Adaptors](#928-channel-adaptors)
      - [(1) 功能](#1-%E5%8A%9F%E8%83%BD-6)
      - [(2) 例子](#2-%E4%BE%8B%E5%AD%90-5)
    - [9.2.9 Endpoint](#929-endpoint)
  - [9.3 Demo项目：Email Integration Flow](#93-demo%E9%A1%B9%E7%9B%AEemail-integration-flow)
      - [(1) 项目及功能介绍](#1-%E9%A1%B9%E7%9B%AE%E5%8F%8A%E5%8A%9F%E8%83%BD%E4%BB%8B%E7%BB%8D)
      - [(2) Integration Flow配置](#2-integration-flow%E9%85%8D%E7%BD%AE)
      - [(2) InboundAdaptor实现](#2-inboundadaptor%E5%AE%9E%E7%8E%B0)
        - [(a) Mail Endpoint使用](#a-mail-endpoint%E4%BD%BF%E7%94%A8)
        - [(b) 依赖](#b-%E4%BE%9D%E8%B5%96)
        - [(c) 配置](#c-%E9%85%8D%E7%BD%AE)
      - [(2) Transformer实现](#2-transformer%E5%AE%9E%E7%8E%B0)
      - [(3) 消息处理器的实现](#3-%E6%B6%88%E6%81%AF%E5%A4%84%E7%90%86%E5%99%A8%E7%9A%84%E5%AE%9E%E7%8E%B0)
      - [(4) 禁用`Sprint MVC自动配置`](#4-%E7%A6%81%E7%94%A8sprint-mvc%E8%87%AA%E5%8A%A8%E9%85%8D%E7%BD%AE)
  - [9.4 小结](#94-%E5%B0%8F%E7%BB%93)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->



> * 格式形如`1.2.1`的章节序号为原书的章节序号
> * 格式形如`(1)/(2)/(3)`的章节序号为在笔记中展开的内容

# CH09 Spring Integration

> 很多应用都需要连接外部系统才能工作，这些系统连接准循着一定的模式（参考《Enterprise Inetgration Patterns，Addison-Wesley，2003）。Spring Integration为众多的集成模式提供了实现，每个模式实现为一个组件，借助Spring配置，这些组件被组装成一个管道，消息通过组件在管道中传递。

## 9.1 简单Integration Flow

> 一个简单的Integration  Flow，包含了Spring Inetegration众多的特性和特点
>
> Demo项目位置：[../ch09/simple-flow/](../ch09/simple-flow/)

### (1) 应用程序部分

#### (a) 添加依赖

> [/simple-flow/pom.xml](../ch09/simple-flow/pom.xml)
>
> ~~~xml
> <!-- spring integration starter -->
> <dependency>
>   <groupId>org.springframework.boot</groupId>
>   <artifactId>spring-boot-starter-integration</artifactId>
> </dependency>
> 
> <!-- 创建Integration  Flow，需要声明它能够收发哪些应用程序之外资源的数据 -->
> <!-- 文件系统是其中一种资源 -->
> <!-- 此demo需要从文件系统收发数据，引入文件端点模块 -->
> <dependency>
>   <groupId>org.springframework.integration</groupId>
>   <artifactId>spring-integration-file</artifactId>
> </dependency>
> ~~~

#### (b) 定义消息网关

> 为应用创建一种方法，让它能够发送数据到Integration Flow中，进而通过Integration Flow把数据写入到文件中
>
> 代码：[/simple-flow/src/main/java/sia5/FileWriterGateway.java](../ch09/simple-flow/src/main/java/sia5/FileWriterGateway.java)
>
> ~~~java
> // 集成流提供给应用程序的消息网关，应用程序通过该网关把数据发送到Flow中，并让Flow执行相应的操作
> // * @MessageingGateway   ：让Spring Integration在运行时生成该接口的实现
> // * defaultRequestChannel：接口内方法调用时所形成的消息要发送给指定的channel
> @MessagingGateway(defaultRequestChannel="textInChannel")
> public interface FileWriterGateway {
>     // @Header(FileHeaders.FILENAME)表示参数filename的值应该包含在消息头中，而不是消息载荷中
>     void writeToFile(@Header(FileHeaders.FILENAME) String filename, String data);
> }
> ~~~
>
> 消息网关是Integration Flow与应用程序交互连接点，网关定义好之后，就可以配置Integration Flow了。共有三种配置方法：（1）XML；（2）Java配置；（3）DSL（Domain Specific Language）的Java配置

#### (c) 将数据发送到消息网关

> ```java
> @SpringBootApplication
> public class SimpleFlowApplication {
> 	public static void main(String[] args) {
> 		SpringApplication.run(SimpleFlowApplication.class, args);
> 	}
> 
>     @Bean
> 	public CommandLineRunner writeData(FileWriterGateway gateway, Environment env) {
> 		return args -> {
>             // 将数据输出到Gateway
> 			gateway.writeToFile(
>                 "simple.txt", 
>                 "Hello, Spring Integration!");
>         };
> 	}
> }
> ```
>
> 代码：[/simple-flow/src/main/java/sia5/SimpleFlowApplication.java](../ch09/simple-flow/src/main/java/sia5/SimpleFlowApplication.java)

### (2) Integration Flow部分

> 通过网关，应用程序将数据发送给Integration Flow，而Integration Flow如何运行，则需要更为具体的配置。整个Integration Flow的流程图示（使用Enterprise Integration Patterns）如下：
>
> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/spring_in_action_5_integ_simple_flow.jpg)
>
> 下面的三个小节，使用三种不同的方法进行配置

### 9.1.1 使用XML配置

> 较为老式的配置
>
> 在Demo项目的`spring.profile`配置为`"xmlconfig"`时生效

#### (1) 配置内容及说明

> 代码：[/simple-flow/src/main/resources/filewriter-config.xml](../ch09/simple-flow/src/main/resources/filewriter-config.xml)
>
> ~~~xml
> <?xml version="1.0" encoding="UTF-8"?>
> <beans ...>
> 	<!-- 通道：网关FileWriteGateway发送消息的通道 -->
> 	<int:channel id="textInChannel" />
> 
> 	<!-- 转换器：(1)从textInChannel通道读取消息（2）调用SpEL将载荷转为大写（3）转换后的消息写入到fileWriterChannel通道 -->
> 	<int:transformer id="upperCase"
> 		input-channel="textInChannel"
> 		output-channel="fileWriterChannel"
> 		expression="payload.toUpperCase()" />
> 
> 	<!-- 通道：转换器upperCase的输出通道，适配器writer的输入通道 -->
> 	<int:channel id="fileWriterChannel" />
> 
> 	<!-- 出站通道适配器：将从fileWriterChannel读到的数据写入文件 -->
>     <!-- * int-file：Spring Integration文件模块提供的命名空间 -->
>     <!-- * outbound-channel-adapter：表示这是出站通道的适配器 -->
>     <!-- * fileWritterChannel：该适配器的输入通道是fileWritterChannel -->
>     <!-- * 其他属性：写文件操作的配置，其中不包括文件名，文件名由消息头的file_name属性指定 -->
>     <int-file:outbound-channel-adapter id="writer"
>         channel="fileWriterChannel"
>         directory="/tmp/sia5/files"
>         mode="APPEND"
>         append-new-line="true" />
> </beans>
> ~~~

#### (2) 将配置导入Spring应用

> ```java
> @Configuration
> public class FileWriterIntegrationConfig {
>     // 当spring.profiles配置为"xmlconfig"时该配置生效
> 	@Profile("xmlconfig")
> 	@Configuration
> 	@ImportResource("classpath:/filewriter-config.xml")
> 	public static class XmlConfiguration {}
>     
>     // 当spring.profiles配置不是"xmlconfig"时，其他配置不生效
>     // ... (忽略) ...
> }
> ```
>
> 代码：[/simple-flow/src/main/java/sia5/FileWriterIntegrationConfig.java](../ch09/simple-flow/src/main/java/sia5/FileWriterIntegrationConfig.java)

### 9.1.2 使用Java配置

> 在Demo项目的`spring.profile`配置为`"javaconfig"`时生效

#### (1) 配置内容及说明

> ```java
> @Configuration
> public class FileWriterIntegrationConfig {
> 	// 当spring.profiles配置为"javaconfig"时生效
> 
> 	// 两个Channel可以不用显示声明、框架会自动创建，除非想做更细化的配置
> 	// @Profile("javaconfig")
>     // @Bean
> 	// public MessageChannel textInChannel() {
> 	// 	   return new DirectChannel();
> 	// }
> 
> 	// @Profile("javaconfig")
> 	// @Bean
> 	// public MessageChannel fileWriterChannel() {
> 	// 	   return new DirectChannel();
> 	// }
> 
> 	// 转换器：输入为textInChannel、输出为fileWriterChannel
> 	@Profile("javaconfig")
> 	@Bean
> 	@Transformer(inputChannel="textInChannel", outputChannel="fileWriterChannel")
> 	public GenericTransformer<String, String> upperCaseTransformer() {
> 		return text -> text.toUpperCase();
> 	}
> 
>     // 文件写入消息处理器：输入为fileWriteChannel
> 	@Profile("javaconfig")
> 	@Bean
> 	@ServiceActivator(inputChannel="fileWriterChannel")
> 	public FileWritingMessageHandler fileWriter() {
>         // 写入目录是"/tmp/sia5/files"，文件名由消息头指定
> 		FileWritingMessageHandler handler = new FileWritingMessageHandler(new File("/tmp/sia5/files"));
> 		handler.setExpectReply(false);						// 告诉ServiceActivator不要期望会有答复通道
> 		handler.setFileExistsMode(FileExistsMode.APPEND); 	// 追加写
> 		handler.setAppendNewLine(true);						// 从新行开始追加
> 		return handler;
> 	}
> 
> 	// 当spring.profiles配置不是"javaconfig"时，其他配置不生效
> 	// ... (忽略) ...
> }
> ```
>
> 代码：[/simple-flow/src/main/java/sia5/FileWriterIntegrationConfig.java](/simple-flow/src/main/java/sia5/FileWriterIntegrationConfig.java)

### 9.1.3 使用DSL配置

> 在Demo项目的`spring.profile`配置为`"javaconfig"`时生效

#### (1) 配置内容及说明

> 使用DSL将不用把每个组件都声明为单独的bean，而是使用一个bean来定义一整条Integration Flow
>
> ```java
> @Configuration
> public class FileWriterIntegrationConfig {
> 	// 当spring.profiles配置为"javadsl"时生效
> 	@Profile("javadsl")
> 	@Bean
> 	public IntegrationFlow fileWriterFlow() {
> 		return IntegrationFlows
> 			.from(MessageChannels.direct("textInChannel"))	 		 // 不用声明入站通道，直接引用即可，框架会自动创建
> 			.<String, String>transform(t -> t.toUpperCase()) 		 // 转换器
>             // .channel(MessageChannels.direct("fileWriterChannel")) // 可以省略、除非想自定义
> 			.handle(										 		 // 文件写入
>             	Files.outboundAdapter(new File("/tmp/sia5/files")) 
> 					.fileExistsMode(FileExistsMode.APPEND)
> 					.appendNewLine(true))
> 			.get(); // 生成IntegrationFlows对象
> 	}
> 
> 	// 当spring.profiles配置不是"javadsl"时，其他配置不生效
> 	// ... (忽略) ...
> }
> ```
>
> 代码：[/simple-flow/src/main/java/sia5/FileWriterIntegrationConfig.java](/simple-flow/src/main/java/sia5/FileWriterIntegrationConfig.java)

## 9.2 Spring Integration功能概览

### (1) Integration Flow中各个组件的功能 

> Integration Flow是由一个或多个组件组成，功能如下
>
> | 功能     | 组件                            | 功能                                                         |
> | -------- | ------------------------------- | ------------------------------------------------------------ |
> | 通道     | 通道（channel）                 | 将消息从一个元素传递到另一个元素                             |
> | 数据处理 | 过滤器（filter）                | 基于某些断言，条件化地允许某些消息通过流                     |
> |          | 转换器（transformer）           | 改变消息的值和/或将消息载荷从一种类型转换成另一种类型        |
> |          | 路由器（router）                | 将消息路由至一个或多个通道，通常会基于消息的头信息进行路由   |
> |          | 切分期（splitter）              | 将传入的消息切割成两个或者更多的消息，然后每个消息发送至不同的通道 |
> |          | 聚合器（aggregator）            | 切分器的反向操作，将来自不同通道的多个消息合并成一个消息     |
> | 外部接口 | 服务激活器（service activator） | 将消息传递给某个Java方法来进行处理，并将返回值发布到输出通道上 |
> |          | 通道适配器（channel adaptor）   | 将通道连接到某些外部系统或传输方式，可接受输入，也可以写出到外部系统 |
> |          | 网关（gateway）                 | 通过接口将数据传递到Integration Flow中，也可选接收流的结果作为响应 |

###  9.2.1 Message Channel

#### (1) Spring Integration提供的通道实现

> Channel用于Integration Pipline中消息的传递，除了上一节例子中的`DirectChannel`以外，Spring Integration提供如下多种 消息通道实现
>
> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/sia5_springinteg_channel.jpg)
>
> | 通道（channel）实现      | 说明                                                         |
> | ------------------------ | ------------------------------------------------------------ |
> | PublishSubscribeChannel  | 消息会被广播到一个或多个消费者中（每个消费者都能收到）       |
> | DirectChannel (默认通道) | 与PublishSubscribeChannel类似，但是消息只会发送至一个消费者，它会在与发送者相同的线程中调用消费者。这种方式允许事务跨通道。 |
> | ExecutorChannel          | 类似于DirectChannel，但是消息分发是通过TaskExecutor实现的，这样会在与发送者独立的线程中执行。这种通道类型不支持事务跨通道 |
> | QueueChannel             | 消息会存储到一个队列中，会按照先进先出的方式被拉取出来（只会被一个消费者取到） |
> | PriorityChannel          | 与QueueChannel类似，Message Header中的priority字段中值最高的会先被拉取出来 |
> | RendezvousChannel        | 与QueueChannel类似，但是发送者会一直阻塞通道，直到消费者接收到消息为止，实际上会同步发送者和消费者 |
> | FluxMessageChannel       | 反应式流的发布者消息通道，基于Reactor项目的Flux              |

#### (2) 创建和使用

##### (a) 创建

> 当使用Java配置或者DSL配置时，缺省 情况下 `inputChannel="someChannelName"`会自动设成DirectChannel，改变channel类型需要显式地配置MessageChannel Bean，例如：
>
> ~~~java
> @Bean
> public MessageChannel orderChannel() {
> 	return new PublishSubscribeChannel();
> }
> ~~~

##### (b) 使用

> 如果使用Java Configure，可以在消费这个channel上使用@ServiceActivator
>
> ~~~java
> @ServiceActivator(inputChannel="orderChannel")
> ~~~
>
> 如果使用Java DSL，可以通过`channel()`方法来引用
>
> ~~~java
> @Bean
> public IntegrationFlow orderFlow() {
> 	return IntegrationFlows
> 		...
> 		.channel("orderChannel")
> 		...
> 		.get();
> }
> ~~~

##### (c) QueueChannel

> 如果使用QueueChannel，需要为它配置一个轮询器，以一定的间隔轮询该通道
>
> ~~~java
> @Bean
> public MessageChannel orderChannel() {
> 	return new QueueChannel();
> }
> ~~~
>
> ~~~java
> @ServiceActivator(inputChannel="orderChannel", poller=@Poller(fixedRate="1000"))
> ~~~

### 9.2.2 Filter

#### (1) 功能

Filter根据断言决定消息是否可以进入Integration Flow的下一步

> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/sia5_springinteg_filter.jpg)

#### (2) 例子

例子：使用Java Configure定义

> ~~~java
> @Filter(inputChannel="numberChannel",  outputChannel="evenNumberChannel")
> public boolean evenNumberFilter(Integer number) {
> 	return number % 2 == 0;
> }
> ~~~

例子：使用Java DSL定义

> ~~~java
> @Bean
> public IntegrationFlow evenNumberFlow(AtomicInteger integerSource) {
> 	return IntegrationFlows
> 		...
>         //用lambda表达式实现函数式接口Gener
> 		.<Integer>filter((p) -> p % 2 == 0)
> 		...
> 		.get();
> }
> ~~~

### 9.2.3 Transformer

#### (1) 功能

Transformer对消息进行转换，产生不同的消息，甚至是不同的载荷类型

> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/sia5_springinteg_transformer.jpg)

#### (2) 例子

例子：使用Java Configure定义

> ~~~java
> // 泛型参数分别是输入类型、输出类型
> @Bean
> @Transformer(inputChannel="numberChannel", outputChannel="romanNumberChannel")
> public GenericTransformer<Integer, String> romanNumTransformer() {
>     return RomanNumbers::toRoman;
> }
> ~~~

例子：使用Java DSL定义

> ~~~java
> @Bean
> public IntegrationFlow transformerFlow() {
> 	return IntegrationFlows
> 		...
>         // 用lambda表达式实现函数式接口
> 		.transform(RomanNumbers::toRoman)
> 		...
> 		.get();
> }
> ~~~
>
> ~~~java
> @Bean
> public RomanNumberTransformer romanNumberTransformer() {
>     // 对较为复杂的转换逻辑，也可将其封装在一个类中
>     // 这个类实现了GenericTransformer接口
> 	return new RomanNumberTransformer();
> }
> 
> @Bean
> public IntegrationFlow transformerFlow(RomanNumberTransformer romanNumberTransformer) {
> 	return IntegrationFlows
> 		...
> 		.transform(romanNumberTransformer)
> 		...
> 		.get();
> }
> ~~~

### 9.2.4 Router

#### (1) 功能

Router基于某个断言，实现集成流的分治，将消息发送到不同的通道上 

> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/sia5_springinteg_router.jpg)

#### (2) 例子

例子：使用Java Configure定义

> ~~~java
> // 定义router
> @Bean
> @Router(inputChannel="numberChannel")
> public AbstractMessageRouter evenOddRouter() {
> 	return new AbstractMessageRouter() {
>         // 根据Message决定将其发送到哪些channel上
> 		@Override
> 		protected Collection<MessageChannel> determineTargetChannels(Message<?> message) {
>             Integer number = (Integer) message.getPayload();
>             if (number % 2 == 0) {
>                 return Collections.singleton(evenChannel());
>             } else {
>                 return Collections.singleton(oddChannel());
>             }
> 		}
> 	};
> }
> 
> // 相关的channel
> @Bean
> public MessageChannel evenChannel() {
>   return new DirectChannel();
> }
> 
> @Bean
> public MessageChannel oddChannel() {
>   return new DirectChannel();
> }
> ~~~

例子：使用Java DSL定义

> ~~~java
> @Bean
> public IntegrationFlow numberRoutingFlow(AtomicInteger source) {
> 	return IntegrationFlows
> 		...
> 		.<Integer, String>route(
>         	// 路由判断逻辑 
>         	n -> n%2==0 ? "EVEN":"ODD", 
> 			// 根据判断结果执行不同的sub flow
>         	mapping -> mapping.subFlowMapping("EVEN", 
> 						sf -> sf.<Integer, Integer>transform(n -> n * 10)
> 								.handle((i,h) -> { ... })
> 					).subFlowMapping("ODD", 
> 						sf -> sf.transform(RomanNumbers::toRoman)
> 								.handle((i,h) -> { ... }))
> 		).get();
> }
> ~~~

### 9.2.5 Splitter

#### (1) 功能

Splitter可以将一个消息切分成多个消息独立处理，这些消息也可以讲给不同的sub-flow

> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/sia5_springinteg_splitter.jpg)

#### (2) 典型应用场景

> 1. 载荷是一个列表（例如是一个商品列表），希望将这个列表拆开（例如希望对每件商品单独处理）
> 2. 载荷是一组有关联的信息（例如购买订单包含了投递信息、账单、商品项等信息），希望不同的信息交给不同的子流来处理（通常会在splitter之后再配一个router来把拆分后的消息映射到不同的子流上）

#### (3) 例子

公用代码：用来拆分订单信息的类

> ~~~java
> public class OrderSplitter {
> 	public Collection<Object> splitOrderIntoParts(PurchaseOrder po) {
> 		ArrayList<Object> parts = new ArrayList<>();
> 		parts.add(po.getBillingInfo());
> 		parts.add(po.getLineItems());
> 		return parts;
> 	}
> }
> ~~~

例子（场景2 → 场景1）：使用Java Configure定义

> 定义Splitter（场景2）
>
> ~~~java
> // 通过@Splitter将其声明为Integration Flow中的一部分，并标注为@Bean使其可以被注入
> @Bean
> @Splitter(inputChannel="poChannel", outputChannel="splitOrderChannel")
> public OrderSplitter orderSplitter() {
> 	return new OrderSplitter();
> }
> ~~~
>
> 定义Router（串行在Splitter之后）
>
> ~~~java
> // 使用@Router注解和@Bean注解
> @Bean
> @Router(inputChannel="splitOrderChannel")
> public MessageRouter splitOrderRouter() {
>     // PayloadTypeRouter会根据载荷数据类型来进行路由
> 	PayloadTypeRouter router = new PayloadTypeRouter();
>     // 载荷类型为BillingInfo：路由到billingInfoChannel通道
>     // 载荷类型为List：路由到listItemChannel通道
> 	router.setChannelMapping(BillingInfo.class.getName(), "billingInfoChannel");
> 	router.setChannelMapping(List.class.getName(), "lineItemsChannel");
> 	return router;
> }
> ~~~
>
> 定义Splitter（场景 1）
>
> ~~~java
> // 使用@Splitter注解
> @Bean
> @Splitter(inputChannel="lineItemsChannel", outputChannel="lineItemChannel")
> public List<LineItem> lineItemSplitter(List<LineItem> lineItems) {
> 	return lineItems;
> }
> ~~~

例子：使用Java DSL

> ~~~java
> return IntegrationFlows
> 	...
> 	// 定义splitter
> 	.split(orderSplitter())
> 	// 定义router
> 	.<Object, String> route(
> 		// 参数1：路由断言
> 		p -> {
> 			return (p.getClass().isAssignableFrom(BillingInfo.class)) ? "BILLING_INFO":"LINE_ITEMS";} 
> 		}, 
> 		// 参数2：基于断言执行结果的sub-flow映射
> 		mapping -> mapping
> 			.subFlowMapping(
>                 // 账单的sub-flow映射
> 				"BILLING_INFO", 
> 				sf -> sf
> 					 .<BillingInfo> handle((billingInfo, h) -> { ... })
> 			)
> 			.subFlowMapping(
>                 // Item的sub-flow映射
> 				"LINE_ITEMS", 
> 				sf -> sf
> 					 .split() // 链表切分为元素
> 					 .<LineItem> handle((lineItem, h) -> {...})
> 			)
> 		)
> 	）.get();
> ~~~

### 9.2.6 Service Activators

#### (1) 功能

> * 作为集成流的终点，接收来自channel的消息，通过MessageHandler来调用某个外部服务处理消息中的数据
>
> * 作为集成流的一个节点，接收来自channel的消息，通过GenericHandler处理这个消息，随后将其转发给下游channel
>
> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/sia5_springinteg_svcactivator.jpg)

#### (2) 例子

例子：使用Java Configuration，作为集成流的终点

> ~~~java
> @Bean
> @ServiceActivator(inputChannel="someChannel")
> public MessageHandler sysoutHandler() {
> 	// 使用 MessageHandler 函数式接口
>     // 节点没有 outputChannel，是集成流的终点
> 	return message -> {
> 		System.out.println("Message payload:  " + message.getPayload());
> 	};
> }
> ~~~

例子：使用Java Configuration，作为集成流的中间节点

> ~~~java
> @Bean
> @ServiceActivator(inputChannel="orderChannel", outputChannel="completeOrder")
> public GenericHandler<Order> orderHandler(OrderRepository orderRepo) {
>     // 使用 GenericHandler<Order> 函数式接口
>     // 该lambda表达式被框架调用执行完毕之后，会将输入的Order对象发送到”completeOrder“通道中
> 	return (payload, headers) -> {
>         // GenericHandler不仅能拿到payload，还能拿到headers
> 		return orderRepo.save(payload);
> 	};
> }
> ~~~

例子：使用Java DSL，作为集成流的终点

> ~~~java
> public IntegrationFlow someFlow() {
> 	return IntegrationFlows
> 		...
>         // 使用handle方法接受MessageHandler接口的实现类对象，作为流的终点
> 		.handle(msg -> {
> 			System.out.println("Message payload:  " + msg.getPayload());
> 		})
> 		.get();
> }
> ~~~

例子：使用Java DSL，作为集成流的中间节点

> ~~~java
> public IntegrationFlow orderFlow(OrderRepository orderRepo) {
> 	return IntegrationFlows
> 		...
>         // 使用handle方法接受GenericHandler<InputPayloadType>的实现类对象，作为流的中间节点
> 		.<Order>handle((payload, headers) -> {
> 			return orderRepo.save(payload);
> 		})
> 		...
> 		.get();
> }
> ~~~
>
> 如果`GenericHandler<InputPayloadType>`被用作集成流的终点、它必须`return null`否则会报`"no output channel specified"`的错误

### 9.2.7 Gateway

#### (1) 功能 

> 通过Gateway、应用程序可以提交数据到集成流中，同时也可选接收流的结果作为响应。
>
>  ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/sia5_springinteg_gateways.jpg)

#### (2) 例子

例子：使用 Java Configure，只提交数据不接收响应

> ~~~java
> // 只需声明接口不需要实现，Spring会为标记了@messagingGateway生成实现代码
> @MessagingGateway(defaultRequestChannel="textInChannel")
> public interface FileWriterGateway {
>     // 方法返回void，没有replay channel
>     void writeToFile(
>         // @Header(FileHeaders.FILENAME)表示参数filename的值应该包含在消息头中，而不是消息载荷中    
>         @Header(FileHeaders.FILENAME) String filename, 
>         String data
>     );
> }
> ~~~

例子：在package scan范围内声明一个@Component，提交数据并接收响应

> ~~~java
> package com.example.demo;
> import org.springframework.integration.annotation.MessagingGateway;
> import org.springframework.stereotype.Component;
> 
> // 只需声明接口不需要实现，Spring会为标记了@messagingGateway生成实现代码
> // * 应用通过inChannel发布到集成流中，传给方法
> // * 方法的返回值通过outChannel返回给应用
> @Component
> @MessagingGateway(defaultRequestChannel="inChannel", defaultReplyChannel="outChannel")
> public interface UpperCaseGateway {
> 	String uppercase(String in);
> }
> ~~~

例子：使用Java DSL配置

> 创建一个集成流（IntegrationFlow）就相当于定义了GateWay，不需要另外单独声明
>
> ~~~java
> @Bean
> public IntegrationFlow uppercaseFlow() {
> 	return IntegrationFlows
> 		.from("inChannel") 		// 输入通道
> 		.<String, String> transform(s -> s.toUpperCase()) 
> 		.channel("outChannel")	// 输出通道
> 		.get();
> }
> ~~~

### 9.2.8 Channel Adaptors

#### (1) 功能 

> 代表集成流的`入口`（入站通道适配器，使用`@InboundChannelAdaptor`注解，或DSL中的`from`方法）和`出口`（出站通道适配器，使用`@OutboundChannelAdaptor`注解）
>
> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/sia5_springinteg_channel_adapters.jpg)
>
> 关于入站适配器：形式非常丰富，如下面的例子
>
> 关于出现适配器：通常会把ServiceActivator用作出站通道适配器

#### (2) 例子

例子：把一个AtomicInteger自增器用作入站适配器

> 使用Java Confi编写
>
> ~~~java
> @Bean
> // 入站适配器
> @InboundChannelAdapter(
>     poller=@Poller(fixedRate="1000"), // 每1000毫秒调用一次MessageSource<Integer>的receive方法
>     channel="numberChannel"			  // 方法调用后返回的Message传入numberChannel通道
> )
> public MessageSource<Integer> numberSource(AtomicInteger source) {
> 	// 用lambda表达式实现函数式接口
>     return () -> {
>         // MessageSource<Integer>.receive()方法的实现
>         // 为一个AtomicInteger自增并返回自增后的值
> 		return new GenericMessage<>(source.getAndIncrement());
> 	};
> }
> ~~~
>
> 使用Java DSL编写
>
> ~~~java
> @Bean
> public IntegrationFlow someFlow(AtomicInteger integerSource) {
> 	return IntegrationFlows
>         // 输入适配器
> 		.from(
> 			integerSource, 		// 注入进来参数，类型是AtomicInteger
> 			"getAndIncrement",	// 调用该AtomicInteger的getAndIncrement方法
> 			c -> c.poller(Pollers.fixedRate(1000)) // 轮询器，每1000毫秒执行一次
> 		)
> 		...
> 		.get();
> }
> ~~~

例子：监控一个目录，并将写入目录的文件以消息的形式提交到file-channel通道中

> 使用 Java Config编写
>
> ~~~java
> @Bean
> @InboundChannelAdapter(
> 	channel="file-channel",	// 调用FileReadingMessageSource.receive方法得到的返回值发送到"file-channel"中
> 	poller=@Poller(fixedDelay="1000")	// 每1000毫秒调用一次MessageSource<T>的receive方法
> )
> public MessageSource<File> fileReadingMessageSource() {
>     // 不必像上一个例子中实现MessageSource<T>接口，使用框架提供类
> 	FileReadingMessageSource sourceReader = new FileReadingMessageSource();
> 	sourceReader.setDirectory(new File(INPUT_DIR));
> 	sourceReader.setFilter(new SimplePatternFileListFilter(FILE_PATTERN));
> 	return sourceReader;
> }
> ~~~
>
> `MessageSource<T>`接口的定义
>
> ```java
> @FunctionalInterface
> public interface MessageSource<T> {
> 	Message<T> receive();
> }
> ```
>
> 使用Java DSL实现
>
> ~~~java
> @Bean
> public IntegrationFlow fileReaderFlow() {
> 	return IntegrationFlows
>         // 创建输入适配器
> 		.from(
>         	// Files
>         	Files.inboundAdapter(new File(INPUT_DIR)).patternFilter(FILE_PATTERN)
>     	)
> 		.get();
> }
> ~~~

### 9.2.9 Endpoint

Spring Integration提供了20多个包含通道适配器的`Endpoint模块`

对于每一个`Endpoint模块`

> * 可以在Java Config中将其声明为bean
> * 可以在Java DSL配置中通过静态方法的方式来引用
>
> 在下一小节中，会演示其中`Email模块`（`spring-integration-mail`）的使用

模块列表如下表

> | Module                    | Dependency artifact ID (Group ID: org.springframework.integration) |
> | ------------------------- | ------------------------------------------------------------ |
> | AMQP                      | spring-integration-amqp                                      |
> | Spring application events | spring-integration-event                                     |
> | RSS and Atom              | spring-integration-feed                                      |
> | Filesystem                | spring-integration-file                                      |
> | FTP/FTPS                  | spring-integration-ftp                                       |
> | GemFire                   | spring-integration-gemfire                                   |
> | HTTP                      | spring-integration-http                                      |
> | JDBC                      | spring-integration-jdbc                                      |
> | JPA                       | spring-integration-jpa                                       |
> | JMS                       | spring-integration-jms                                       |
> | Email                     | spring-integration-mail                                      |
> | MongoDB                   | spring-integration-mongodb                                   |
> | MQTT                      | spring-integration-mqtt                                      |
> | Redis                     | spring-integration-redis                                     |
> | RMI                       | spring-integration-rmi                                       |
> | SFTP                      | spring-integration-sftp                                      |
> | STOMP                     | spring-integration-stomp                                     |
> | Stream                    | spring-integration-stream                                    |
> | Syslog                    | spring-integration-syslog                                    |
> | TCP/UDP                   | spring-integration-ip                                        |
> | Twitter                   | spring-integration-twitter                                   |
> | Web Services              | spring-integration-ws                                        |
> | WebFlux                   | spring-integration-webflux                                   |
> | WebSocket                 | spring-integration-websocket                                 |
> | XMPP                      | spring-integration-xmpp                                      |
> | ZooKeeper                 | spring-integration-zookeeper                                 |

## 9.3 Demo项目：Email Integration Flow

#### (1) 项目及功能介绍

> 实现一个集成流，它会：
>
> 1. 轮询taco订单Email的收件箱
> 2. 解析Email为订单对象
> 3. 将订单提交到系统的REST API中
>
> 步骤3也可以直接调用repository的方式来进行、但不论使用哪种，Integration Flow都不会发生改变，改变的只是Service Activator的实现方式而已
>
> 流程图示如下：
>
> ![](https://raw.githubusercontent.com/kenfang119/pics/main/002_spring_boot/sia5_springinteg_emailstreamsample.jpg)
>
> Demo项目位置：[ch09/taco-cloud/](../ch09/taco-cloud/)

#### (2) Integration Flow配置

> 使用Java DSL的方式来实现
>
> 代码：[/taco-cloud/tacocloud-email/src/main/java/tacos/email/TacoOrderEmailIntegrationConfig.java](../ch09/taco-cloud/tacocloud-email/src/main/java/tacos/email/TacoOrderEmailIntegrationConfig.java)
>
> ~~~java
> @Configuration
> public class TacoOrderEmailIntegrationConfig {
> 	@Bean
> 	public IntegrationFlow tacoOrderEmailFlow(
> 			// 用于创建InboundAdapter的属性配置
> 			EmailProperties emailProps, 
> 			// Transformer<Order>用于从email中提取Order数据的Transformer
> 			EmailToOrderTransformer emailToOrderTransformer, 
> 			// GenericHandler<Order>传递给Service Activator用作OutboundAdapter
> 			// 处理从上游channel收到的订单对象，并将其提交至Taco Cloud的Rest API
> 			OrderSubmitMessageHandler orderSubmitHandler 
>     ) {
> 		return IntegrationFlows
> 				// Inbound Adapter
> 				.from(
> 					// 使用Mail Enpoint提供的静态方法创建InboundAdaptor
> 					Mail.imapInboundAdapter(
> 						emailProps.getImapUrl()),	 // IMP URL配置
> 						e -> e.poller(Pollers.fixedDelay(
> 							emailProps.getPollRate() // 轮询间隔配置
> 						))
> 				)
> 				// Transformer
> 				.transform(emailToOrderTransformer)
> 				// Outbound Adapter (Service Activator)
> 				.handle(orderSubmitHandler)
> 				.get();
> 	}
> }
> ~~~

#### (2) InboundAdaptor实现

##### (a) Mail Endpoint使用 

> ```java
> // 使用Spring Integration提供的Email Endpoint的静态方法创建Inbound Adaptor
> Mail.imapInboundAdapter(emailProps.getImapUrl() /*IMP URL*/) 
> ```

##### (b) 依赖

> ```xml
> <dependency>
> 	<groupId>org.springframework.integration</groupId>
> 	<artifactId>spring-integration-mail</artifactId>
> </dependency>
> ```

##### (c) 配置

> 使用configuration properties编写的配置类：[/taco-cloud/tacocloud-email/src/main/java/tacos/email/EmailProperties.java](../ch09/taco-cloud/tacocloud-email/src/main/java/tacos/email/EmailProperties.java)
>
> ```java
> @Data
> @ConfigurationProperties(prefix="tacocloud.email")
> @Component
> public class EmailProperties {
> 	private String username;
> 	private String password;
> 	private String host;
> 	private String mailbox;
> 	private long pollRate = 30000;
>   
> 	public String getImapUrl() {
> 		return String.format("imaps://%s:%s@%s/%s",
> 			this.username, this.password, this.host, this.mailbox);
> 	}
> }
> ```
>
> 配置：[/taco-cloud/tacocloud-email/src/main/resources/application.yml](../ch09/taco-cloud/tacocloud-email/src/main/resources/application.yml)
>
> ```yml
> tacocloud:
> 	email:
> 		host: imap.tacocloud.com
> 		mailbox: INBOX
> 		username: taco-in-flow
> 		password: 1L0v3T4c0s
> 		poll-rate: 10000
> ```

#### (2) Transformer实现

> 完整代码：[/taco-cloud/tacocloud-email/src/main/java/tacos/email/EmailToOrderTransformer.java](../ch09/taco-cloud/tacocloud-email/src/main/java/tacos/email/EmailToOrderTransformer.java)
>
> ~~~java
> // 创建成一个Bean，以便能够注入到TacoOrderEmailIntegrationConfig中 
> @Component
> // 基类AbstractMailMessageTransformer可以抽取Email的信息并将其放入Message对象中，以传递给doTransform方法
> public class EmailToOrderTransformer extends AbstractMailMessageTransformer<Order> {
> 	@Override
> 	protected AbstractIntegrationMessageBuilder<Order> doTransform(Message mailMessage) throws Exception {
> 		// 业务逻辑，从Message中提取数据创建Order对象
> 		Order tacoOrder = processPayload(mailMessage);
>         // 把Order对象用作载荷，生成Message返回
> 		return MessageBuilder.withPayload(tacoOrder);
> 	}
> 	...
> }
> ~~~
>
> 用于封装业务数据的代码（Order、Taco、Ingredient）：属性 + 用lambok生成的getter/setter方法
>
> * [/taco-cloud/tacocloud-email/src/main/java/tacos/email/Order.java](../ch09/taco-cloud/tacocloud-email/src/main/java/tacos/email/Order.java)
> * [/taco-cloud/tacocloud-email/src/main/java/tacos/email/Taco.java](../ch09/taco-cloud/tacocloud-email/src/main/java/tacos/email/Taco.java)
> * [/taco-cloud/tacocloud-email/src/main/java/tacos/email/Ingredient.java](../ch09/taco-cloud/tacocloud-email/src/main/java/tacos/email/Ingredient.java)

#### (3) 消息处理器的实现

> 用于帮助框架生成Service Activator，用作Outbound Adaptor
>
> 完整代码：[/taco-cloud/tacocloud-email/src/main/java/tacos/email/OrderSubmitMessageHandler.java](../ch09/taco-cloud/tacocloud-email/src/main/java/tacos/email/OrderSubmitMessageHandler.java)
>
> ~~~java
> @Component
> public class OrderSubmitMessageHandler implements GenericHandler<Order> {
> 	private RestTemplate rest;
> 	private ApiProperties apiProps; 
> 	public OrderSubmitMessageHandler(ApiProperties apiProps, RestTemplate rest) {
> 		this.apiProps = apiProps;
> 		this.rest = rest;
> 	}
> 
> 	@Override
> 	public Object handle(Order order, Map<String, Object> headers) {
>         // 发送Rest请求，将Order对象提交给远端的Rest Service
> 		rest.postForObject(apiProps.getUrl(), order, String.class);
> 		return null;
> 	}
> }
> ~~~
>
> Configuration Properties类：[/taco-cloud/tacocloud-email/src/main/java/tacos/email/ApiProperties.java](../ch09/taco-cloud/tacocloud-email/src/main/java/tacos/email/ApiProperties.java)
>
> ~~~java
> @Data
> @ConfigurationProperties(prefix="tacocloud.api")
> @Component
> public class ApiProperties {
> 	private String url;
> }
> ~~~
>
> 配置：[/taco-cloud/tacocloud-email/src/main/resources/application.yml](../ch09/taco-cloud/tacocloud-email/src/main/resources/application.yml)
>
> ~~~yml
> tacocloud:
> 	api:
> 		url: http://localhost:8080/orders/fromEmail
> ~~~
>
> 依赖：[/taco-cloud/tacocloud-email/pom.xml](../ch09/taco-cloud/tacocloud-email/pom.xml)
>
> 为了使用RestTemplate，引入spring-boot-starter-web
>
> ```xml
> <dependency>
> 	<groupId>org.springframework.boot</groupId>
> 	<artifactId>spring-boot-starter-web</artifactId>
> </dependency>
> ```

#### (4) 禁用`Sprint MVC自动配置`

> 依赖`spring-boot-starter-web`会使Spring Boot自动配置Spring MVC及嵌入式tomcat（并不需要）
>
> 因此在[application.yml](../ch09/taco-cloud/tacocloud-email/src/main/resources/application.yml)中添加如下配置来禁用Spring MVC自动配置，使其成为独立的Sprint Integration流
>
> ```yml
> spring:
>   main:
>     # 可选值为servlet（默认值）, reactive, none
>     web-application-type: none 
> ```

## 9.4 小结

> 略