借鉴了RocketMQ和Kafka的：
    顺序append文件，提供很好的性能
    顺序消费文件，使用offset表示消费进度，成本极低
    将所有subject的消息合并在一起，减少parition数量，可以提供更多的subject(RocketMQ)
    