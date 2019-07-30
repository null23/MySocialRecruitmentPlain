https://mp.weixin.qq.com/s/IxS46JAr7D9sBtCDr8pd7A

#### 1）消费端弄丢了数据

唯一可能导致消费者弄丢数据的情况，就是说，你那个消费到了这个消息，然后消费者那边自动提交了offset，让kafka以为你已经消费好了这个消息，其实你刚准备处理这个消息，你还没处理，你自己就挂了，此时这条消息就丢咯。
这不是一样么，大家都知道kafka会自动提交offset，那么只要关闭自动提交offset，在处理完之后自己手动提交offset，就可以保证数据不会丢。但是此时确实还是会重复消费，比如你刚处理完，还没提交offset，结果自己挂了，此时肯定会重复消费一次，自己保证幂等性就好了。
生产环境碰到的一个问题，就是说我们的kafka消费者消费到了数据之后是写到一个内存的queue里先缓冲一下，结果有的时候，你刚把消息写入内存queue，然后消费者会自动提交offset。
然后此时我们重启了系统，就会导致内存queue里还没来得及处理的数据就丢失了

#### 2）kafka弄丢了数据
这块比较常见的一个场景，就是kafka某个broker宕机，然后重新选举partiton的leader时。大家想想，要是此时其他的follower刚好还有些数据没有同步，结果此时leader挂了，然后选举某个follower成leader之后，他不就少了一些数据？这就丢了一些数据啊。
生产环境也遇到过，我们也是，之前kafka的leader机器宕机了，将follower切换为leader之后，就会发现说这个数据就丢了

所以此时一般是要求起码设置如下4个参数：
    给这个topic设置replication.factor参数：这个值必须大于1，要求每个partition必须有至少2个副本
    在kafka服务端设置min.insync.replicas参数：这个值必须大于1，这个是要求一个leader至少感知到有至少一个follower还跟自己保持联系，没掉队，这样才能确保leader挂了还有一个follower吧
    在producer端设置acks=all：这个是要求每条数据，必须是写入所有replica之后，才能认为是写成功了
    在producer端设置retries=MAX（很大很大很大的一个值，无限次重试的意思）：这个是要求一旦写入失败，就无限重试，卡在这里了
    我们生产环境就是按照上述要求配置的，这样配置之后，至少在kafka broker端就可以保证在leader所在broker发生故障，进行leader切换时，数据不会丢失

#### 3）生产者会不会弄丢数据
    如果按照上述的思路设置了ack=all，一定不会丢，要求是，你的leader接收到消息，所有的follower都同步到了消息之后，才认为本次写成功了。如果没满足这个条件，生产者会自动不断的重试，重试无限次。

### 一个线上的offset丢失的问题处理
kafka将某消费组的offset删除，导致consuner重新消费（取决于auto.offset.reset）

1、如果在低流量的分区上存在consumer，则kafka可能会删除该consumer已经提交的偏移量。一旦删除了偏移量，消费者的重新启动或者重新balance将导致消费者找不到任何提交的偏移量，并从最早/最晚开始消费（取决于auto.offset.reset）这会导致只允许消费一次的消息有可能会被消费多次

目前的解决方法是：
1、定期为您拥有的每个分区提交一个偏移量，确保它比offsets.retention.minutes（消费者可能不知道的代理端设置）小
2、将offsets.retention.minutes的值变得非常高。您必须确保它高于您要支持的任何有效的低流量速率。例如，如果您想支持某人每月生产一次的主题，则必须将offsetes.retention.mintues设置为1个月。
3、打开enable.auto.commit（这基本上是＃1，但更容易实现）

offsets.retention.minutes的值可改变kafka offset的过期时间，单位为分钟。默认是1440 也就是一天，我们kafka的消息过期时间是7天，因此我们只需将时间设为10080即可