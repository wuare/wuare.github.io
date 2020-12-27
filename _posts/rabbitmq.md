RabbitMQ基本概念
Message,Publisher,Exchange,Binding,Queue,Channel,Consumer,Virtual Host
<!-- more -->

Message：消息
Publisher：消息生产者，将消息推送到Exchange，消费者从Queue中获取
Exchange：交换器，用来接收生产者的消息并将这些消息路由给服务器中的队列
Binding：绑定，用于给Exchange和Queue建立关系，从而决定将这个交换器中的哪些消息，发送到对应的Queue
Queue：消息队列，用来保存消息知道发送给消费者
Consumer：消费者
Virtual Host：虚拟主机，表示一批交换器、消息队列和相关对象

Exchange支持的四种策略
Direct策略
消息中的路由键（routing key）如果和Binding中的binding key一致，交换器就将消息发送到对应的队列中
eg：Exchange 和两个队列绑定在一起
Q1 的 bindingkey 是 orange
Q2 的 binding key 是 black 和 green.
当 Producer 发布一个消息，其routing key是orange时, exchange 会把它放到 Q1 上, 如果是black或green就会到 Q2 上, 其余的 Message 被丢弃

Topic策略
通过routing key与 bingding key的模式匹配方式来分发消息
简单来讲，直接策略是完全精确匹配，而topic则支持正则匹配，满足某类指定规则的（如以 xxx 开头的路由键），可以将消息分发过去
`#` 匹配 0 个或多个单词
`*` 匹配不多不少一个单词
eg：
Producer发送消息时需要设置routing_key,
Q1 的 binding key 是*.orange.*
Q2 是 *.*.rabbit 和 lazy.#：
发布一个routing key为test.orange.mm 消息，则会路由到 Q1；
注意： 如果是routng key是 test.orange则无法路由到 Q1，
因为 Q1 的规则是三个单词，中间一个为 orange，不满足这个规则的都无效
发布一个routing key为test.qq.rabbit或者lazy.qq的消息 都可以分发到 Q2；即路由 key 为三个单词，最后一个为 rabbit 或者不限制单词个数，主要第一个是 lazy 的消息，都可以分发过来
如果发布的是一个test.orange.rabbit消息，则 Q1 和 Q2 都可以满足
注意： 这时两个队列都会接受到这个消息

Fanout策略
广播策略，忽略routing key 和 binding key，将消息分发给所有绑定在这个 exchange 上的 queue

Headers策略
根据 Message 的一些头部信息来分发过滤 Message，忽略 routing key 的属性，如果 Header 信息和 message 消息的头信息相匹配