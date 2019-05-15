1.producer->broker使用mysql的事务，同一个mysql实例是可以保证事务的
2.broker->consumer通过ack确认码，只要刷盘成功，就ok了
3.qmq很多是依赖dubbo的