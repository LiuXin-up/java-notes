# 查看版本对应：[RabbitMQ Erlang 版本要求](https://www.rabbitmq.com/which-erlang.html)

# Erlang

## 下载Erlang安装程序

1.  访问Erlang官方网站（[https://www.erlang.org/downloads](https://www.erlang.org/downloads）。)）
2.  选择合适的安装程序，并下载到本地

## 安装Erlang

1.  运行下载的Erlang安装程序进行安装。

2.  配置环境变量
    1.  创建 ERLANG\_HOME

        ![在这里插入图片描述](https://img-blog.csdnimg.cn/4968adb143424091ab555e90f1ed0542.png "在这里插入图片描述")
    2.  添加环境变量到path

        ![在这里插入图片描述](https://img-blog.csdnimg.cn/6537a1b0ff7b4f0aaa0c601fc0c92cd2.png "在这里插入图片描述")
    3.  查看是否安装成功

        在cmd中，执行命令 **erl**。显示如下，则安装成功

        ```bash
        C:\Users\liuxin>erl
        Erlang/OTP 26 [erts-14.0.2] [source] [64-bit] [smp:12:12] [ds:12:12:10] [async-threads:1] [jit:ns]
        Eshell V14.0.2 (press Ctrl+G to abort, type help(). for help)
        ```

# RabbitMQ

## 下载RabbitMQ安装程序

1.  访问RabbitMQ官方网站（[https://www.rabbitmq.com/download.html）](https://www.rabbitmq.com/download.html）。)
2.  在"Windows"部分下载适用于Windows系统的安装程序。

## 安装RabbitMQ

1.  运行下载的RabbitMQ安装程序。在选择安装目录时，将RabbitMQ安装在一个没有空格或特殊字符的路径中。

2.  安装时，选择将RabbitMQ作为window服务。启动命令如下：
    ```bash
    # 启动mq
    net start RabbitMQ
    # 停止mq服务
    net stop RabbitMQ
    # 重启mq
    net restart RabbitMQ
    ```

3.  若启动时，报错 \*\*\*`Please set ERLANG_SERVICE_MANAGER_PATH to the folder containing "erlsrv.exe"`  \*\*\*等信息，则需要配置一下环境变量。

    将报错提示的环境变量名称添加到系统变量中，如上述报错 \*\*\*请将ERLANG\_SERVICE\_MANAGER\_PATH设置为包含“erlsrv.exe”的文件夹  \*\*\*

    则在变量中，添加 ***`ERLANG_SERVICE_MANAGER_PATH`  的环境变量，并将变量地址指向*** ...\Erlang OTP\erts-14.0.2\bin 这个包含“erlsrv.exe”的文件夹

4.  访问mq的管理页面：

    RabbitMQ默认禁用管理界面，需要通过命令开启管理界面

    以管理员模式打开 cmd窗口，进入RabbitMQ安装目录的**sbin**文件夹路径下，执行`rabbitmq-plugins enable rabbitmq_management`命令，然后重启RabbitMQ

5.  进入管理页面

    默认账号：guest\
    默认密码：guest

# Spring Boot 集成RabbitMQ

## 引入依赖

在pom依赖中，引入以下依赖。由于项目中使用的是spring boot 2.3.7，所以引入相同版本依赖。

         <!--    RabbitMQ    -->
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-starter-amqp</artifactId>
                <version>2.3.7.RELEASE</version>
            </dependency>

## 配置yml文件

    # RabbitMQ
    spring:
      rabbitmq:
        host: 127.0.0.1
        port: 5672
        username: user
        password: user

## 发送消息

### service层

使用 `RabbitTemplate` 会默认将消息进行持久化

```java
// ---- 省略 import、方法名、自动注入、controller层 等代码 ---- //  	
	@Autowired
    private RabbitTemplate rabbitTemplate;

    /**
     * 队列的名称
     */
    String queueName = "testQueue";

    /**
     * 发送消息
     *
     * @param message 要发送的消息
     */
    public String sendMessage(String message) {
        try {
            RabbitAdmin rabbitAdmin = new RabbitAdmin(rabbitTemplate);
            //如果queueName队列不存在，创建队列
            if (Objects.isNull(rabbitAdmin.getQueueProperties(queueName))) {
                org.springframework.amqp.core.Queue queue = new Queue(queueName);
                rabbitAdmin.declareQueue(queue);
            }
            rabbitTemplate.convertAndSend(queueName, message);
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
        return "消息发送成功";
    }
```

### 查看消息是否成功发送

在rabbitmq的管理页面 --> **Queues and Streams**(队列和流) 中，查看**Messages**列的信息，若发送成功，则ready、total会+1

<table>
    <tr> 
    	<th colspan="5">Overview</th>
        <th colspan="3">Messages</th>
		<th colspan="3">Message rates</th>
   </tr>
   <tr> 
		<td>Virtual host</td>
   		<td>Name</td>
   		<td>Type</td>
   		<td>Features</td>
   		<td>State</td>
   		<td>Ready</td>
   		<td>Unacked</td>
   		<td>Total</td>
   		<td>incoming</td>
   		<td>deliver / get</td>
   		<td>ack</td>
   </tr>
    <tr> 
		<td>/</td>
   		<td>testQueue</td>
   		<td>	classic</td>
   		<td>D</td>
   		<td>idle</td>
   		<td>1</td>
   		<td>0</td>
   		<td>1</td>
   		<td>0.00/s</td>
   		<td>0.00/s</td>
   		<td>0.00/s</td>
   </tr>
</table>

## 接收消息

### service层

```java
// ---- 省略 import、方法名、自动注入、controller层 等代码 ---- //

    /**
     * 从队列中获取消息
     */
    public String receiveMessage() {
        Object message = rabbitTemplate.receiveAndConvert(queueName);
        System.out.println("Received message: " + message);
        return message.toString();
    }
```

### 查看消息是否成功接收

在rabbitmq的管理页面 --> **Queues and Streams**(队列和流) 中，查看**Messages**列的信息，若发送成功，则ready、total会-1

<table>
    <tr> 
    	<th colspan="5">Overview</th>
        <th colspan="3">Messages</th>
		<th colspan="3">Message rates</th>
   </tr>
   <tr> 
		<td>Virtual host</td>
   		<td>Name</td>
   		<td>Type</td>
   		<td>Features</td>
   		<td>State</td>
   		<td>Ready</td>
   		<td>Unacked</td>
   		<td>Total</td>
   		<td>incoming</td>
   		<td>deliver / get</td>
   		<td>ack</td>
   </tr>
    <tr> 
		<td>/</td>
   		<td>testQueue</td>
   		<td>	classic</td>
   		<td>D</td>
   		<td>idle</td>
   		<td>0</td>
   		<td>0</td>
   		<td>0</td>
   		<td>0.00/s</td>
   		<td>0.00/s</td>
   		<td>0.00/s</td>
   </tr>
</table>

## 查看队列中未被消费的消息数量

```java
// ---- 省略 import、方法名、自动注入、controller层 等代码 ---- //
    
	/**
     * 查询未被消费的条数
     */
    public long getUnconfirmedCount() throws IOException, TimeoutException {
        Connection connection = new ConnectionFactory().newConnection();
        Channel channel = connection.createChannel();
        // 获取未被消费的消息数量
        long unacknowledgedMessages = channel.messageCount(queueName);
        System.out.println("未被消费的消息数量：" + unacknowledgedMessages);
        return unacknowledgedMessages;
    }
```

