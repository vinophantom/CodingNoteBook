# RabbitMQ



## AMQP

Advanced Message Queuing Protocol.

用于统一面向消息中间件实现的一套标准协议，其设计目标是对于*消息*的排序、路由（包括点对点和订阅-发布）、保持可靠性、保证安全性。高级消息队列协议保证了由不同提供商发行的客户端之间的互操作性。与先前的中间件标准（如Java消息服务），在特定的API接口层面和实现行为进行了统一不同，高级消息队列协议关注于各种消息如何作为字节流进行传递。因此，使用了符合协议实现的任意应用程序之间可以保持对消息的创建、传递。

###  

## MQ
Message Queue.
是一种应用程序对应用程序的通信方法。应用程序通过读写出入队列的消息（针对应用程序的数据）来通信，而无需专用连接来链接它们。
消息传递指的是程序之间通过在消息中发送数据进行通信，而不是通过直接调用彼此来通信。队列的使用除去了接收和发送应用程序同时执行的要求。
在项目中，将一些无需即时返回且耗时的操作提取出来，进行了异步处理，而这种异步处理的方式大大的节省了服务器的请求响应时间，从而提高了系统的吞吐量。

## Main Concept

***Broker***：消息队列服务器实体。
***Exchange***：消息交换机，它指定消息按什么规则，路由到哪个队列。
***Queue***：消息队列载体，每个消息都会被投入到一个或多个队列。
***Binding***：绑定，它的作用就是把exchange和queue按照路由规则绑定起来。
***Routing Key***：路由关键字，exchange根据这个关键字进行消息投递。
***vhost***：虚拟主机，一个broker里可以开设多个vhost，用作不同用户的权限分离。
***producer***：消息生产者，就是投递消息的程序。
***consumer***：消息消费者，就是接受消息的程序。
***channel***：消息通道，在客户端的每个连接里，可建立多个channel，每个channel代表一个会话任务。



### Exchange

| Name             | Default pre-declared names |
| ---------------- | -------------------------- |
| Direct Exchange  | empty OR amq.direct        |
| Fanout Exchange  | amq.fanout                 |
| Topic Exchange   | amq.topic                  |
| Headers Exchange | amq.match                  |

