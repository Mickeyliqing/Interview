# Redis 基础知识

## Redis 基础数据结构

**string**

1. 使用场景

存储 key-value 键值对。

1. 常用命令

```shell
set   [key]  [value]   给指定key设置值（set 可覆盖老的值）
get  [key]   获取指定key 的值
del  [key]   删除指定key
exists  [key]  判断是否存在指定key
mset  [key1]  [value1]  [key2]  [value2] ...... 批量存键值对
mget  [key1]  [key2] ......   批量取key
expire [key]  [time]    给指定key 设置过期时间  单位秒
setex    [key]  [time]  [value]  等价于 set + expire 命令组合
setnx  [key]  [value]   如果key不存在则set 创建，否则返回0
incr   [key]           如果value为整数 可用 incr命令每次自增1
incrby  [key] [number]  使用incrby命令对整数值 进行增加 number
```



**list**

1. 使用场景

消息队列。

1. 常用命令

```shell
rpush  [key] [value1] [value2] ......    链表右侧插入
rpop    [key]  移除右侧列表头元素，并返回该元素
lpop   [key]    移除左侧列表头元素，并返回该元素
llen  [key]     返回该列表的元素个数
lrem [key] [count] [value]  删除列表中与value相等的元素，count是删除的个数。 count>0 表示从左侧开始查找，删除count个元素，count<0 表示从右侧开始查找，删除count个相同元素，count=0 表示删除全部相同的元素
(PS:   index 代表元素下标，index 可以为负数， index= 表示倒数第一个元素，同理 index=-2 表示倒数第二 个元素。)
lindex [key] [index]  获取list指定下标的元素 （需要遍历，时间复杂度为O(n)）
lrange [key]  [start_index] [end_index]   获取list 区间内的所有元素 （时间复杂度为 O（n））
ltrim  [key]  [start_index] [end_index]   保留区间内的元素，其他元素删除（时间复杂度为 O（n））
```



**hash**

1. 使用场景

系统对象的存储。

1. 常用命令

```shell
hset  [key]  [field] [value]    新建字段信息
hget  [key]  [field]    获取字段信息
hdel [key] [field]  删除字段
hlen  [key]   保存的字段个数
hgetall  [key]  获取指定key 字典里的所有字段和值 （字段信息过多,会导致慢查询 慎用：亲身经历 曾经用过这个这个指令导致线上服务故障）
hmset  [key]  [field1] [value1] [field2] [value2] ......   批量创建
hincr  [key] [field]   对字段值自增
hincrby [key] [field] [number] 对字段值增加number
```



**set**

1. 使用场景

需要存放的数据不能重复。

1. 常用命令

```shell
sadd  [key]  [value]  向指定key的set中添加元素
smembers [key]    获取指定key 集合中的所有元素
sismember [key] [value]   判断集合中是否存在某个value
scard [key]    获取集合的长度
spop  [key]   弹出一个元素
srem [key] [value]  删除指定元素
```



**zset**

1. 使用场景

需要对数据根据某个权重进行排序的场景。

1. 常用命令

```shell
zadd [key] [score] [value] 向指定key的集合中增加元素
zrange [key] [start_index] [end_index] 获取下标范围内的元素列表，按score 排序输出
zrevrange [key] [start_index] [end_index]  获取范围内的元素列表 ，按score排序 逆序输出
zcard [key]  获取集合列表的元素个数
zrank [key] [value]  获取元素再集合中的排名
zrangebyscore [key] [score1] [score2]  输出score范围内的元素列表
zrem [key] [value]  删除元素
zscore [key] [value] 获取元素的score
```



## Redis 设置过期时间

设置过期时间：Expire 。

移除过期时间：Persist。

过期键删除策略：

1. 立即删除。设置键的过期时间时，创建一个回调事件，当过期时间到达，有回调事件删除。
2. 惰性删除。每次从 Dict 字典按 Key 取值时，先检查 Key 是否已经过期，如果过期了在删除它。
3. 定时删除。每隔一段时间对 Expires 字典进行检查，删除里面的过期键。

实际使用为：惰性删除 + 定期删除策略。



## Redis 内存淘汰策略

1. 从已设置过期时间的数据集中选择最近最少使用的数据淘汰。
2. 从已设置过期时间的数据集中选择将要过期的数据淘汰。
3. 从已设置过期时间的数据集中选择任意数据淘汰。
4. 移除最近最少使用的 Key 。
5. 从数据集中任意选择数据集淘汰。



## Redis 常见的三种问题

- 缓存穿透

用户查询数据库不存在的数据，数据也不会在缓存中存储。当用户发起请求，它永远不会访问缓存，数据库压力就会增大。

**解决方案：**

1. 参数检验，上层拦截。
2. 查询结果为空也做缓存，但有效期设置较短，避免影响正常数据的使用。



- 缓存击穿

热点数据存储到期，多个线程同时请求热点数据，缓存刚好过期，所有并发都会访问数据库。

**解决方案：**

1. 设置热点键。



- 缓存雪崩

数据未加载到缓存或者缓存大范围失效，导致请求查询数据库。

**解决方案：**

1. 事前：高可用缓存。
2. 事中：缓存降级。
3. 事后：Redis 备份和快速预热。



## Redis 实现分布式锁

**加锁**

1. 使用 setnx 命令保证互斥性。

    setnx key value

2. 需要设置锁的过期时间。

    setnx key seconds value

    expire key seconds 

3. setnx 和设置过期时间需要保证原子性，避免设置 setnx 成功之后在设置过期时间客户端奔溃导致死锁。

4. 加锁 Value 值作为唯一标识，可以采用 UUID 作为唯一标识，加锁成功后需要把唯一标识返回给客户端进行解锁操作。

5. jedis.set(String key, String value, String nxxx, String expx, int time)，这个set()方法一共有五个形参：  

    第一个为key，我们使用key来当锁，因为key是唯一的。  

    第二个为value，我们传的是requestId，很多童鞋可能不明白，有key作为锁不就够了吗，为什么还要用到value？原因就是我们在上面讲到可靠性时，分布式锁要满足第四个条件解铃还须系铃人，通过给value赋值为requestId，我们就知道这把锁是哪个请求加的了，在解锁的时候就可以有依据。requestId可以使用UUID.randomUUID().toString()方法生成。  

    第三个为nxxx，这个参数我们填的是NX，意思是SET IF NOT EXIST，即当key不存在时，我们进行set操作；若key已经存在，则不做任何操作；  

    第四个为expx，这个参数我们传的是PX，意思是我们要给这个key加一个过期的设置，具体时间由第五个参数决定。  

    第五个为time，与第四个参数相呼应，代表key的过期时间。 

    

**解锁**

1. 需要拿加锁成功的唯一标识进行解锁，从而保证加锁和解锁的是同一个客户端。
2. 解锁操作需要比较唯一标识是否相等，相等在执行删除操作。

（这两个操作可以使用 Lua 脚本方式来实现）



## Redis 实现分布式锁遇到的问题

1. 锁未释放，导致线程池满了。

    **解决方案：**释放掉对应的锁。

2. Setnx 命令原理：当 Key 不存在时将 Key 的值设置成 Value，返回值 1。若给定的 Key 已经存在，则 Setnx 不做任何操作，返回值为 0。

    **解决方案：**加锁的时候传递一个参数，将这个参数作为 Key 的 Value。每次解锁的时候判断 Value 是否相等即可。

3. 数据库事务，获取锁的等待时间远超数据库事务的超时时间，程序就会报异常。

    **解决方案：**将数据库事务设置成手动提交，回滚事务。

4. 锁过期了，业务还没执行完。

    **解决方案：**调用 redisson 方法的 API 。redisson 在加锁成功后，会注册一个定时任务监听这个锁，每隔 10 秒就去查看这个锁，如果还持有锁，就对过期时间进行续期。默认过期时间 30 秒。

5. 主从复制。集群环境下，master 已经加锁成功，并已经复制给 slave 节点，当宕机了后，完成主备切换后，新的请求在新的 master 继续加锁。这就导致同一时间内，多个请求对分布式锁完成加锁，导致脏数据产生。

    **解决方案：**暂无。



