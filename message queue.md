message queue

In [computer science](https://en.wikipedia.org/wiki/Computer_science), **message queues** and **mailboxes** are [software-engineering](https://en.wikipedia.org/wiki/Software_engineering) [components](https://en.wikipedia.org/wiki/Software_componentry) typically used for [inter-process communication](https://en.wikipedia.org/wiki/Inter-process_communication) (IPC), or for inter-[thread](https://en.wikipedia.org/wiki/Thread_(computing)) communication within the same process. They use a [queue](https://en.wikipedia.org/wiki/Queue_(data_structure)) for [messaging](https://en.wikipedia.org/wiki/Message_(computer_science)) – the passing of control or of content. [Group communication systems](https://en.wikipedia.org/wiki/Group_communication_system) provide similar kinds of functionality.

The message queue paradigm is a sibling of the [publisher/subscriber](https://en.wikipedia.org/wiki/Publish–subscribe_pattern) pattern, and is typically one part of a larger [message-oriented middleware](https://en.wikipedia.org/wiki/Message-oriented_middleware) system. Most messaging systems support both the publisher/subscriber and message queue models in their [API](https://en.wikipedia.org/wiki/Application_programming_interface), e.g. [Java Message Service](https://en.wikipedia.org/wiki/Java_Message_Service) (JMS).



The message queue software can be either proprietary, open source or a mix of both. It is then on run either on premise in private servers or on external cloud servers ([message queuing service](https://en.wikipedia.org/wiki/Message_queuing_service)).

- Proprietary options have the longest history, and include products from the inception of message queuing, such as [IBM MQ](https://en.wikipedia.org/wiki/IBM_MQ), and those tied to specific operating systems, such as [Microsoft Message Queuing (MSMQ)](https://en.wikipedia.org/wiki/Microsoft_Message_Queuing). Cloud service providers also provide their proprietary solutions such as [Amazon Simple Queue Service](https://en.wikipedia.org/wiki/Amazon_Simple_Queue_Service) (SQS), [StormMQ](https://en.wikipedia.org/wiki/StormMQ), [Solace](https://en.wikipedia.org/wiki/Solace_Corporation), and [IBM MQ](https://en.wikipedia.org/wiki/IBM_MQ).
- Open source choices of messaging [middleware](https://en.wikipedia.org/wiki/Middleware) systems includes [Apache ActiveMQ](https://en.wikipedia.org/wiki/Apache_ActiveMQ), [Apache Kafka](https://en.wikipedia.org/wiki/Apache_Kafka), [Apache Qpid](https://en.wikipedia.org/wiki/Apache_Qpid), [Apache RocketMQ](https://en.wikipedia.org/wiki/Apache_RocketMQ), [Enduro/X](https://en.wikipedia.org/wiki/Enduro/X), [JBoss Messaging](https://en.wikipedia.org/wiki/JBoss_Messaging), [JORAM](https://en.wikipedia.org/wiki/JORAM), [RabbitMQ](https://en.wikipedia.org/wiki/RabbitMQ), [Sun Open Message Queue](https://en.wikipedia.org/wiki/Open_Message_Queue), and [Tarantool](https://en.wikipedia.org/wiki/Tarantool).



消息队列的三个特点(相对于http式通信方式)

1异步2通信双方的解耦3限流









RPC 使用rabbitmq

时间通知使用kafka



amqp

