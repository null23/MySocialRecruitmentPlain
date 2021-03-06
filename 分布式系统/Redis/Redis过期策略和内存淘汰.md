### 过期策略
#### 定期删除
    redis默认是每隔100ms就随机抽取一些设置了过期时间的key，检查其是否过期，如果过期就删除。假设redis里放了10万个key，都设置了过期时间，你每隔几百毫秒，就检查10万个key，那redis基本上就死了，cpu负载会很高的，消耗在你的检查过期key上了。注意，这里可不是每隔100ms就遍历所有的设置过期时间的key，那样就是一场性能上的灾难。实际上redis是每隔100ms随机抽取一些key来检查和删除的。

#### 惰性删除
    在你获取某个key的时候，redis会检查一下 ，这个key如果设置了过期时间那么是否过期了？如果过期了此时就会删除，不会给你返回任何东西。
    并不是key到时间就被删除掉，而是你查询这个key的时候，redis再懒惰的检查一下。


### 内存淘汰
    如果redis的内存占用过多的时候，此时会进行内存淘汰，内存淘汰策略有以下几个

#### noeviction
    当内存不足以容纳新写入数据时，新写入操作会报错，这个一般没人用吧，实在是太恶心了
#### allkeys-lru
    当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key（这个是最常用的）
#### allkeys-random
    当内存不足以容纳新写入数据时，在键空间中，随机移除某个key，这个一般没人用吧，为啥要随机，肯定是把最近最少使用的key给干掉啊
#### volatile-lru
    当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key（这个一般不太合适）
#### volatile-random
    当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key
#### volatile-ttl
    当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除