### 缓存雪崩
事前：Redis高可用
    1.主从+哨兵
    2.redis cluster

事中
    1.本地缓存
    2.Hystrix限流

事后
    Redis持久化，重启的时候从磁盘重新读数据刷到缓存里

### 缓存穿透
    1.mysql查不到，就往redis对应的值写成null
    2.布隆过滤器