前言：
**
Redis是一种可以基于内存级别的，也可以持久化的，k-v类型的数据库。
说到内存级别的存储，我们难免想到"缓存"。基于内存级别的缓存，在Java中，还有例如Guava Cache等，我们甚至可以用ConcurrentHashMap实现一个缓存。
虽然都是内存级别的缓存，但是Redis和例如Guava Cache等仍然是有区别的，Guava Cache是基于Java的，也就是Guava Cache可以缓存的数据量的大小和JVM中可用的内存大小有关。
在数据量较大的时候，我们就应该选择Redis作为我们缓存。

Jedis：Jedis中封装了一些Java API，通过这些Java API，我们可以和Redis进行交互。

------------------------------------------------------------------------------------------------------------------------------------------------------------
Redis的五种数据结构：
**
1.String 
String是Redis中最基本的数据类型，一个key对应一个value，主要是几个set的API的使用方式，例如：
set(cacheKey)
jedis.set(cacheKey, value, NX_OR_XX, EX_OR_PX, expireTime) 
	NX_OR_XX：NX代表当前key不存在时设置value，XX代表当前key存在时设置value。
	EX_OR_PX：EX代表过期时间单位是秒，PX是毫秒。
	当为一个不存在的key设置过期时间的时候，使用这个API。
jedis.setbit(cacheKey, offset, string/boolean)
	对一个key所存储的字符串的值，进行偏移量上的更改(这个key理应是之前已经存在了的) 相关知识：offset偏移量，bitmap位图。

2.Hash
Redis中的hash是一个String类型的key和filed和value的映射表(注意理解这句话，不能把filed-value理解成一个String，而是key和(filed-value)整体是一个stringString)
可以参照String来理解Hash的一些API，最主要的就是
hset(key, field, value)

3.List
List一个key对应多个value集合，按照插入顺序排序。添加元素的时候，我们可以选择从头添加，也可以从尾添加(可以看做是一个双端队列)。
当然最主要的还是List的API的使用，由于可以从头/从尾添加，我们也可以从头/从尾弹出元素。例如：

对头操作的：
lpush(key, value1, value2...)   从队头插入一个或多个值，key不存在时创建一个新的key，key存在的时候，但是这个key不是List类型，返回一个错误
lpushx(key, value1, value2...)  和lpush不同的是，key不存在的时候，不会创建一个新的key
lpop(key)  从队头弹出一个元素，若没有元素，返回null
blpop(time, key)  从队头弹出一个元素，若没有元素，就阻塞到过期时间time或者有新元素入队

对尾操作的：
rpush
rpushx
rpop
brpop

公共操作：
lindex(index)  获取index位置的元素，没有就返回null。index正数的时候是从头到尾累加的索引值，index负数的时候是从尾到头的索引值。
lset(key, index, value)  设置替换某个索引的值，当索引溢出或者该索引不存在的时候，返回一个错误。
ltrim(key, start, end)  截取start到end索引区间的值