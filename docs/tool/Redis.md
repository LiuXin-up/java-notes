## 安装

#### windwos

1. windows版本是在github上获取。

   下载地址：[https://github.com/microsoftarchive/redis/tags](https://links.jianshu.com/go?to=https%3A%2F%2Fgithub.com%2Fmicrosoftarchive%2Fredis%2Ftags)

2. 进入页面后我们选择最新版本，点击download

3. 点击download后，选择zip版本进行下载，直接解压后即可使用

4. 解压后进入相应目录， 在目录下进入cmd

   输入： `redis-server.exe redis.windows.conf` 进行启动

    `redis-server.exe redis.windows.conf`  --通过指定配置文件进行启动redis

   

#### Linux

1. 打开终端，使用以下命令下载Redis的压缩包：

   ```bash
   wget http://download.redis.io/releases/redis-x.x.x.tar.gz
   ```


   注意将 "x.x.x" 替换为你想要下载的Redis版本号。

2. 解压下载的压缩包

   ```bash
   tar xzf redis-x.x.x.tar.gz
   ```

   进入解压后的目录：

   ```bash
   cd redis-x.x.x
   ```

3. 编译和安装Redis：

   ```bash
   make
   make install
   ```

4. 安装完成后，进入Redis安装目录

   ```bash
   cd /usr/local/bin
   ```

5. 前台启动Redis服务器

   ```bash
   ./redis-server
   ```

   在后台运行

   ```bash
   ./redis-server --daemonize yes
   ```

   Redis默认监听端口为6379。如果修改端口，可以编辑配置文件`redis.conf`进行相应的配置。

6. 检查Redis是否成功运行，可以使用以下命令连接到Redis服务器：

   ```bash
   ./redis-cli
   ```



## 配置

#### 使用redis 需要给redis设置访问密码

1. 打开Redis服务所在的配置文件redis.conf。
2. 配置密码，找到requirepass（若没有则自行添加），并在此选项下添加密码。例如，设置密码为123456，可以这样写：requirepass 123456。
3. 保存配置文件并重启Redis服务即可。

> 推荐使用 **Redis Plus** 可视化工具 进行查看数据

## 数据类型
<span style="font-size: 18px;">String</span>、
<span style="font-size: 18px;">List</span>、
<span style="font-size: 18px;">Hash</span>、
<span style="font-size: 18px;">Set</span>、
<span style="font-size: 18px;">Zset</span>
</br>

#### String 

> 这是最简单的类型，就是普通的 set 和 get，做简单的 KV 缓存。

#### List

> Lists 是有序列表
>
> 通过 list 存储一些列表型的数据结构，例如粉丝列表、文章的评论列表
>
> 用于简单的消息队列

#### Hash

> Hash类型，也叫散列，其value是一个无序字典，类似于Java中的HashMap结构

#### Set

> Sets 是无序集合，自动去重
>
> set 可以进行交集、并集、差集的操作，例如可以把两个人的粉丝列表做一个交集，查看俩人的共同好友是谁

#### Zset

> Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。
>
> 不同之处是有序集合的每个成员都关联了一个评分（score）,这个评分（score）被用来按照从最低分到最高分的方式排序集合中的成员。
>
> 集合的成员是唯一的，但是评分是可以重复的 。
>
> 因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序来获取一个范围的元素



## 缓存的使用

#### 为什么要使用缓存？

> **高性能**
>
> 对于一些需要复杂操作耗时查出来的结果，且确定后面不怎么变化，但是有很多读请求，那么直接将查询出来的结果放在缓存中，后面直接读缓存就好，减少读取的耗时。
>
> **高并发**
>
> mysql 单机支撑到 `2000QPS` 会进行报警。
>
> redis单机支撑的并发量轻松一秒几万甚至十几万，单机承载并发量是 mysql 单机的几十倍。
>
> 缓存是走内存的，内存天然就支撑高并发。

#### 缓存的后果

> 缓存与数据库双写不一致
> 缓存雪崩、缓存穿透、缓存击穿
> 缓存并发竞争



## 过期策略

Redis 过期策略是：**定期删除+惰性删除**。

##### 定期删除

每隔一段时间，redis随机抽取一些设置了过期时间的key，删除掉过期的key。

##### 惰性删除

获取key的时候，检查key是否过期，过期则删除key。

定期删除+惰性删除不能满足需求时（内存不足时），可以走**内存淘汰机制**

##### 内存淘汰机制

- noeviction: 当内存不足以容纳新写入数据时，新写入操作会报错。
- **allkeys-lru**：当内存不足以容纳新写入数据时，在**键空间**中，移除最近最少使用的 key（这个是**最常用**的）。
- allkeys-random：当内存不足以容纳新写入数据时，在**键空间**中，随机移除某个 key。
- volatile-lru：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，移除最近最少使用的 key（这个一般不太合适）。
- volatile-random：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，**随机移除**某个 key。
- volatile-ttl：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，有**更早过期时间**的 key 优先移除。



## Reids三种模式

> 主从、哨兵、集群

- 主从：主节点写数据，从节点读取数据。从节点挂掉不影响读取，主节点挂掉会进行主备切换，保证高可用。
- 哨兵：对主从结构进行监控，主节点挂掉之后，通过投票选举出新的主节点。哨兵模式一般节点为奇数个。
- 集群：对数据进行分片，将数据分到多个节点，减低内存负担。



## 持久化方式

**RDB**(默认)：每隔一段时间，将redis中的数据生成数据快照保存到rdb文件中。

**AOF**：触发设定的条件时，将redis执行的操作指令存储到aof文件中。



## 雪崩、穿透、击穿

#### 雪崩

> 请求量特别大的时候，使用缓存抗住大部分请求，少部分请求访问数据库。此时缓存挂掉，导致全部请求访问数据库，造成数据库崩掉。

**解决方案**

1. 使用Redis集群，主从+哨兵，避免Redis集群全部挂掉
2. 使用部分本地缓存，对接口进行限流，避免数据库崩掉
3. 对Redis进行持久化处理，便于Redis重启后的数据恢复

#### 穿透

> 请求的数据，在缓存中查询不到，在数据库中也查询不到。导致每次请求时都会查询数据库。

**解决方案**

1. 将查询不到的数据建立一个key-value。value值设为null，避免下次查询再访问数据库。
2. 使用布隆过滤器，将请求结果的可能范围映射到布隆过滤器中。请求的key不存在与过滤器中，直接返回不存在。存在与过滤器中，再去查询。

#### 击穿

> 某个key经常被访问，在key失效的瞬间，大量的请求直接打到数据库，缓存失去了作用。

**解决方案**

1. 延长key的失效时间，必要时设为永不过期
2. 定时更新key的失效时间，使用分布式锁，避免更新缓存时数据冲突