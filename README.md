# 什么是消息队列

消息（Message）是指在应用间传送的数据。消息可以非常简单，比如只包含文本字符串，也可以更复杂，可能包含嵌入对象。

消息队列（Message Queue）是一种应用间的通信方式，消息发送后可以立即返回，由消息系统来确保消息的可靠传递。消息发布者只管把消息发布到 MQ 中而不用管谁来取，消息使用者只管从 MQ 中取消息而不管是谁发布的。这样发布者和使用者都不用知道对方的存在。

消息队列是一种应用间的异步协作机制，同时消息队列中间件是分布式系统中重要的组件，主要解决应用耦合，异步消息，流量削锋等问题。实现高性能，高可用，可伸缩和最终一致性架构。是大型分布式系统不可缺少的中间件



消息队列的运用场景

异步处理

场景说明：用户注册后，需要发注册邮件和注册短信。传统的做法有两种1.串行的方式；2.并行方式。

（1）串行方式：将注册信息写入数据库成功后，发送注册邮件，再发送注册短信。以上三个任务全部完成后，返回给客户端。



（2）并行方式：将注册信息写入数据库成功后，发送注册邮件的同时，发送注册短信。以上三个任务完成后，返回给客户端。



1）串行与并行的区别？

假设三个业务节点每个使用50毫秒钟，不考虑网络等其他开销，则串行方式的时间是150毫秒，并行的时间可能是100毫秒。

因为CPU在单位时间内处理的请求数是一定的，假设CPU1秒内吞吐量是100次。则串行方式1秒内CPU可处理的请求量是7次（1000/150）。并行方式处理的请求量是10次（1000/100）。

小结：并行的方式可以提高处理的时间。

2）如以上案例描述，传统的方式系统的性能（并发量，吞吐量，响应时间）会有瓶颈。如何解决这个问题呢？

引入消息队列，将不是必须的业务逻辑，异步处理。改造后的架构如下：



按照以上约定，用户的响应时间相当于是注册信息写入数据库的时间，也就是50毫秒。注册邮件，发送短信写入消息队列后，直接返回，因此写入消息队列的速度很快，基本可以忽略，因此用户的响应时间可能是50毫秒。因此架构改变后，系统的吞吐量提高到每秒20 QPS。比串行提高了3倍，比并行提高了两倍。

小结：引入消息队列 大大提高传统方式的系统性能

应用解耦

场景说明：用户下单后，订单系统需要通知库存系统。传统的做法是，订单系统调用库存系统的接口。如下图：



传统模式的缺点：

1） 假如库存系统无法访问，则订单减库存将失败，从而导致订单失败；

2） 订单系统与库存系统耦合；

如何解决以上问题呢？引入应用消息队列后的方案，如下图：



订单系统：用户下单后，订单系统完成持久化处理，将消息写入消息队列，返回用户订单下单成功。
库存系统：订阅下单的消息，采用拉/推的方式，获取下单信息，库存系统根据下单信息，进行库存操作。
假如：在下单时库存系统不能正常使用。也不影响正常下单，因为下单后，订单系统写入消息队列就不再关心其他的后续操作了。实现订单系统与库存系统的应用解耦
流量削锋

流量削锋也是消息队列中的常用场景，一般在秒杀或团抢活动中使用广泛。

应用场景：

秒杀活动，一般会因为流量过大，导致流量暴增，应用挂掉。为解决这个问题，一般需要在应用前端加入消息队列。

1）可以控制活动的人数；

2）可以缓解短时间内高流量压垮应用；



处理方式：

1）用户的请求，服务器接收后，首先写入消息队列。假如消息队列长度超过最大数量，则直接抛弃用户请求或跳转到错误页面；

2）秒杀业务根据消息队列中的请求信息，再做后续处理

日志处理

日志处理是指将消息队列用在日志处理中，比如Kafka的应用，解决大量日志传输的问题。架构简化如下：


日志采集客户端，负责日志数据采集，定时写受写入Kafka队列；
Kafka消息队列，负责日志数据的接收，存储和转发；
日志处理应用：订阅并消费kafka队列中的日志数据；
以下是新浪kafka日志处理应用案例：



(1) Kafka：接收用户日志的消息队列。

(2) Logstash：做日志解析，统一成JSON输出给Elasticsearch。

(3) Elasticsearch：实时日志分析服务的核心技术，一个schemaless，实时的数据存储服务，通过index组织数据，兼具强大的搜索和统计功能。

(4) Kibana：基于Elasticsearch的数据可视化组件，超强的数据可视化能力是众多公司选择ELK stack的重要原因。

消息通讯

消息通讯是指，消息队列一般都内置了高效的通信机制，因此也可以用在纯的消息通讯。比如实现点对点消息队列，或者聊天室等。

点对点通讯：



客户端A和客户端B使用同一队列，进行消息通讯。

聊天室通讯：



客户端A，客户端B，客户端N订阅同一主题，进行消息发布和接收，实现类似聊天室效果

消息中间件案例

1、电商系统示例



消息队列采用高可用，可持久化的消息中间件。比如Active MQ，Rabbit MQ，Rocket Mq ，kafka

（1）应用将主干逻辑处理完成后，写入消息队列。消息发送是否成功可以开启消息的确认模式。（消息队列返回消息接收成功状态后，应用再返回，这样保障消息的完整性）

（2）扩展流程（发短信，配送处理）订阅队列消息。采用推或拉的方式获取消息并处理。

（3）消息将应用解耦的同时，带来了数据一致性问题，可以采用最终一致性方式解决。比如主数据写入数据库，扩展应用根据消息队列，并结合数据库方式实现基于消息队列的后续处理。

2、日志收集系统示例



消息队列采用高可用，可持久化的消息中间件。比如Active MQ，Rabbit MQ，Rocket Mq ，kafka

（1）应用将主干逻辑处理完成后，写入消息队列。消息发送是否成功可以开启消息的确认模式。（消息队列返回消息接收成功状态后，应用再返回，这样保障消息的完整性）

（2）扩展流程（发短信，配送处理）订阅队列消息。采用推或拉的方式获取消息并处理。

（3）消息将应用解耦的同时，带来了数据一致性问题，可以采用最终一致性方式解决。比如主数据写入数据库，扩展应用根据消息队列，并结合数据库方式实现基于消息队列的后续处理。

MQ的优点和缺点

优点：解耦、削峰、数据分发

缺点包含以下几点：

1）系统可用性降低

系统引入的外部依赖越多，系统稳定性越差。一旦MQ宕机，就会对业务造成影响。
如何保证MQ的高可用？

2）系统复杂度提高

MQ的加入大大增加了系统的复杂度，以前系统间是同步的远程调用，现在是通过MQ进行异步调用。
如何保证消息没有被重复消费？怎么处理消息丢失情况？那么保证消息传递的顺序性？

3）一致性问题

A系统处理完业务，通过MQ给B、C、D三个系统发消息数据，如果B系统、C系统处理成功，D系统处理失败。如何保证消息数据处理的一致性？

该如何选择消息队列

选择消息队列产品的基本标准

虽然这些消息队列产品在功能和特性方面各有优劣，但我们在选择的时候要有一个最低标准，保证入选的产品至少是及格的。

接下来我们先说一下这及格的标准是什么样的。

首先，必须是开源的产品，这个非常重要。开源意味着，如果有一天你使用的消息队列遇到了一个影响你系统业务的Bug，你至少还有机会通过修改源代码来迅速修复或规避这个Bug，解决你的系统火烧眉毛的问题，而不是束手无策地等待开发者不一定什么时候发布的下一个版本来解决。

其次，这个产品必须是近年来比较流行并且有一定社区活跃度的产品。流行的好处是，只要你的使用场景不太冷门，你遇到Bug的概率会非常低，因为大部分你可能遇到的Bug，其他人早就遇到并且修复了。你在使用过程中遇到的一些问题，也比较容易在网上搜索到类似的问题，然后很快的找到解决方案。

还有一个优势就是，流行的产品与周边生态系统会有一个比较好的集成和兼容，比如，Kafka和Flink就有比较好的兼容性，Flink内置了Kafka的Data Source，使用Kafka就很容易作为Flink的数据源开发流计算应用，如果你用一个比较小众的消息队列产品，在进行流计算的时候，你就不得不自己开发一个Flink的Data Source。

最后，作为一款及格的消息队列产品，必须具备的几个特性包括：

●消息的可靠传递：确保不丢消息； ●Cluster：支持集群，确保不会因为某个节点宕机导致服务不可用，当然也不能丢消息； ●性能：具备足够好的性能，能满足绝大多数场景的性能要求

可供选择的几个常见消息队列

RabbitMQ

首先，我们说一下老牌儿消息队列RabbitMQ，俗称兔子MQ。RabbitMQ是使用一种比较小众的编程语言：Erlang语言编写的，它最早是为电信行业系统之间的可靠通信设计的，也是少数几个支持AMQP协议的消息队列之一。

RabbitMQ就像它的名字中的兔子一样：轻量级、迅捷，它的Slogan，也就是宣传口号，也很明确地表明了RabbitMQ的特点：Messaging that just works，“开箱即用的消息队列”。也就是说，RabbitMQ是一个相当轻量级的消息队列，非常容易部署和使用。

另外RabbitMQ还号称是世界上使用最广泛的开源消息队列，是不是真的使用率世界第一，我们没有办法统计，但至少是“最流行的消息中间之一”，这是没有问题的。

RabbitMQ一个比较有特色的功能是支持非常灵活的路由配置，和其他消息队列不同的是，它在生产者（Producer）和队列（Queue）之间增加了一个Exchange模块，你可以理解为交换机。

这个Exchange模块的作用和交换机也非常相似，根据配置的路由规则将生产者发出的消息分发到不同的队列中。路由的规则也非常灵活，甚至你可以自己来实现路由规则。基于这个Exchange，可以产生很多的玩儿法，如果你正好需要这个功能，RabbitMQ是个不错的选择。

RabbitMQ的客户端支持的编程语言大概是所有消息队列中最多的，如果你的系统是用某种冷门语言开发的，那你多半可以找到对应的RabbitMQ客户端。

接下来说下RabbitMQ的几个问题。

第一个问题是，RabbitMQ对消息堆积的支持并不好，在它的设计理念里面，消息队列是一个管道，大量的消息积压是一种不正常的情况，应当尽量去避免。当大量消息积压的时候，会导致RabbitMQ的性能急剧下降。

第二个问题是，RabbitMQ的性能是我们介绍的这几个消息队列中最差的，根据官方给出的测试数据综合我们日常使用的经验，依据硬件配置的不同，它大概每秒钟可以处理几万到十几万条消息。其实，这个性能也足够支撑绝大多数的应用场景了，不过，如果你的应用对消息队列的性能要求非常高，那不要选择RabbitMQ。

最后一个问题是RabbitMQ使用的编程语言Erlang，这个编程语言不仅是非常小众的语言，更麻烦的是，这个语言的学习曲线非常陡峭。大多数流行的编程语言，比如Java、C/C++、Python和JavaScript，虽然语法、特性有很多的不同，但它们基本的体系结构都是一样的，你只精通一种语言，也很容易学习其他的语言，短时间内即使做不到精通，但至少能达到“会用”的水平。

就像一个以英语为母语的人，学习法语、德语都很容易，但是你要是让他去学汉语，那基本上和学习其他这些语言不是一个难度级别的。很不幸的是，Erlang就是编程语言中的“汉语”。所以如果你想基于RabbitMQ做一些扩展和二次开发什么的，建议你慎重考虑一下可持续维护的问题。

RocketMQ

RocketMQ是阿里巴巴在2012年开源的消息队列产品，后来捐赠给 Apache 软件基金会，2017正式毕业，成为Apache的顶级项目。阿里内部也是使用RocketMQ作为支撑其业务的消息队列，经历过多次“双十一”考验，它的性能、稳定性和可靠性都是值得信赖的。作为优秀的国产消息队列，近年来越来越多的被国内众多大厂使用。

RocketMQ就像一个品学兼优的好学生，有着不错的性能，稳定性和可靠性，具备一个现代的消息队列应该有的几乎全部功能和特性，并且它还在持续的成长中。

RocketMQ有非常活跃的中文社区，大多数问题你都可以找到中文的答案，也许会成为你选择它的一个原因。另外，RocketMQ使用Java语言开发，它的贡献者大多数都是中国人，源代码相对也比较容易读懂，你很容易对RocketMQ进行扩展或者二次开发。

RocketMQ对在线业务的响应时延做了很多的优化，大多数情况下可以做到毫秒级的响应，如果你的应用场景很在意响应时延，那应该选择使用RocketMQ。

RocketMQ的性能比RabbitMQ要高一个数量级，每秒钟大概能处理几十万条消息。

RocketMQ的一个劣势是，作为国产的消息队列，相比国外的比较流行的同类产品，在国际上还没有那么流行，与周边生态系统的集成和兼容程度要略逊一筹。

Kafka

最后聊一聊Kafka。Kafka最早是由LinkedIn开发，目前也是Apache的顶级项目。Kafka最初的设计目的是用于处理海量的日志。

在早期的版本中，为了获得极致的性能，在设计方面做了很多的牺牲，比如不保证消息的可靠性，可能会丢失消息，也不支持集群，功能上也比较简陋，这些牺牲对于处理海量日志这个特定的场景都是可以接受的。这个时期的Kafka甚至不能称之为一个合格的消息队列。

但是，请注意，重点一般都在后面。随后的几年Kafka逐步补齐了这些短板，你在网上搜到的很多消息队列的对比文章还在说Kafka不可靠，其实这种说法早已经过时了。当下的Kafka已经发展为一个非常成熟的消息队列产品，无论在数据可靠性、稳定性和功能特性等方面都可以满足绝大多数场景的需求。

Kafka与周边生态系统的兼容性是最好的没有之一，尤其在大数据和流计算领域，几乎所有的相关开源软件系统都会优先支持Kafka。

Kafka使用Scala和Java语言开发，设计上大量使用了批量和异步的思想，这种设计使得Kafka能做到超高的性能。Kafka的性能，尤其是异步收发的性能，是三者中最好的，但与RocketMQ并没有量级上的差异，大约每秒钟可以处理几十万条消息。

使用配置比较好的服务器对Kafka进行过压测，在有足够的客户端并发进行异步批量发送，并且开启压缩的情况下，Kafka的极限处理能力可以超过每秒2000万条消息。

但是Kafka这种异步批量的设计带来的问题是，它的同步收发消息的响应时延比较高，因为当客户端发送一条消息的时候，Kafka并不会立即发送出去，而是要等一会儿攒一批再发送，在它的Broker中，很多地方都会使用这种“先攒一波再一起处理”的设计。当业务场景中，每秒钟消息数量没有那么多的时候，Kafka的时延反而会比较高。所以，Kafka不太适合在线业务场景

我们使用的RabbitMQ相关参数说明

系统配置


配置	类型	默认值	备注
host	string	localhost	Host
port	int	5672	端口号
user	string	guest	用户名
password	string	guest	密码
vhost	string	/	vhost
concurrent.limit	int	0	同时消费的数量
pool	object		连接池配置
pool.connections	int	1	进程内保持的连接数
params	object		基本配置

生产者Producer


参数	类型	备注
exchange	string	交换机类型
routingKey	string	生产者将消息发送给交换机的时候，会指定RoutingKey指定路由规则



交换机类型	说明
直连交换机:Direct	将消息准送到binding key与该消息的routing key相同的队列
主题交换机:Topic	发送到主题交换机的消息不能有任意的routing key,必须是由点号分开的一串单词，这些单词可以是任意的，但通常是与消息相关的一些特性;例如: 路由..key1绑定队列Q1，如：red.blue.key1、black.green.key1等，通过这些路由都可以发送消息给队列Q1 路由#.key2.*绑定队列Q2，如：a.b.c.key2.red、orange.key2.blue、key2.red等，通过这些路由都可以发送消息给队列Q2
扇形交换机:Fanout	生产者发送消息到交换机，交换机再发送到两个队列(广播)

消费者consumer


参数	类型	备注
exchange	string	交换机类型
routingKey	string	生产者将消息发送给交换机的时候，会指定RoutingKey指定路由规则
queue	string	队列:存放供消费者消费的消息
name	string	当前消费者名称
nums	int	处理数量
enable	null	bool	是否禁止进程跟随服务启动
maxConsumption	int	设置此消费者最大处理的消息数，达到指定消费数后，消费者进程会重启

消费结果


返回值	行为
\Hyperf\Amqp\Result::ACK	确认消息正确被消费掉了
\Hyperf\Amqp\Result::NACK	消息没有被正确消费掉，以 basic_nack 方法来响应
\Hyperf\Amqp\Result::REQUEUE	消息没有被正确消费掉，以 basic_reject 方法来响应，并使消息重新入列
\Hyperf\Amqp\Result::DROP	消息没有被正确消费掉，以 basic_reject 方法来响应

对于如何检测消失在传递过程中是否有丢失

宏观方面

总体解决方案

原理非常简单，在Producer端，我们给每个发出的消息附加一个连续递增的序号，然后在Consumer端来检查这个序号的连续性

多消费者解决方案

像Kafka和RocketMQ这样的消息队列，它是不保证在Topic上的严格顺序的，只能保证分区上的消息是有序的，所以我们在发消息的时候必须要指定分区，并且，在每个分区单独检测消息序号的连续性

多生产者解决方案

系统中Producer是多实例的，由于并不好协调多个Producer之间的发送顺序，所以也需要每个Producer分别生成各自的消息序号，并且需要附加上Producer的标识，在Consumer端按照每个Producer分别来检测序号的连续性。

微观方面

生产阶段

在生产阶段，消息队列通过最常用的请求确认机制，来保证消息的可靠传递：当你的代码调用发消息方法时，消息队列的客户端会把消息发送到Broker，Broker收到消息后，会给客户端返回一个确认响应，表明消息已经收到了。客户端收到响应后，完成了一次正常消息的发送。

只要Producer收到了Broker的确认响应，就可以保证消息在生产阶段不会丢失。有些消息队列在长时间没收到发送确认响应后，会自动重试，如果重试再失败，就会以返回值或者异常的方式告知用户。

编写发送消息代码时，需要注意，正确处理返回值或者捕获异常，就可以保证这个阶段的消息不会丢失

存储阶段

在存储阶段正常情况下，只要Broker在正常运行，就不会出现丢失消息的问题，但是如果Broker出现了故障，比如进程死掉了或者服务器宕机了，还是可能会丢失消息的。

如果对消息的可靠性要求非常高，可以通过配置Broker参数来避免因为宕机丢消息。

对于单个节点的Broker，需要配置Broker参数，在收到消息后，将消息写入磁盘后再给Producer返回确认响应，这样即使发生宕机，由于消息已经被写入磁盘，就不会丢失消息，恢复后还可以继续消费。例如，在RocketMQ中，需要将刷盘方式flushDiskType配置为SYNC_FLUSH同步刷盘。

如果是Broker是由多个节点组成的集群，需要将Broker集群配置成：至少将消息发送到2个以上的节点，再给客户端回复发送确认响应。这样当某个Broker宕机时，其他的Broker可以替代宕机的Broker，也不会发生消息丢失。后面我会专门安排一节课，来讲解在集群模式下，消息队列是如何通过消息复制来确保消息的可靠性的。

消费阶段

消费阶段采用和生产阶段类似的确认机制来保证消息的可靠传递，客户端从Broker拉取消息后，执行用户的消费业务逻辑，成功后，才会给Broker发送消费确认响应。如果Broker没有收到消费确认响应，下次拉消息的时候还会返回同一条消息，确保消息不会在网络传输过程中丢失，也不会因为客户端在执行消费逻辑中出错导致丢失。

编写消费代码时需要注意的是，不要在收到消息后就立即发送消费确认，而是应该在执行完所有消费业务逻辑之后，再发送消费确认。

小结

●在生产阶段，你需要捕获消息发送的错误，并重发消息。

●在存储阶段，你可以通过配置刷盘和复制相关的参数，让消息写入到多个副本的磁盘上，来确保消息不会因为某个Broker宕机或者磁盘损坏而丢失。

●在消费阶段，你需要在处理完全部消费业务逻辑之后，再发送消费确认。

理解了这几个阶段的原理后，如果再出现丢消息的情况，应该可以通过在代码中加一些日志的方式，很快定位到是哪个阶段出了问题，然后再进一步深入分析，快速找到问题原因
