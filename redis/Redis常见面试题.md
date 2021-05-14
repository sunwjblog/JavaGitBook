# Redis常见面试题

### 1、缓存穿透

1. #### 什么是缓存穿透

   **key对应的数据在数据源并不存在，每次针对此key的请求从缓存获取不到，请求都会压到数据源，从而可能压垮数据源。**比如用一个不存在的用户id获取用户信息，不论缓存还是数据库都没有，若黑客利用此漏洞进行攻击可能压垮数据库。

   强调的是缓存和数据库都不存在的数据。

   ![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/缓存穿透.png)

2. #### 解决方案

   一个一定不存在缓存及查询不到的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。

   * **对空值缓存：**如果一个查询返回的数据为空（不管是数据是否不存在），我们仍然把这个空结果（null）进行缓存，设置空结果的过期时间会很短，最长不超过五分钟。

   * **设置可访问的名单（白名单）**：使用bitmaps类型定义一个可以访问的名单，名单id作为bitmaps的偏移量，每次访问和bitmap里面的id进行比较，如果访问id不在bitmaps里面，进行拦截，不允许访问。

   * **采用布隆过滤器**：（1） (布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量(位图)和一系列随机映射函数（哈希函数）。

     布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难。)

     将所有可能存在的数据哈希到一个足够大的bitmaps中，一个一定不存在的数据会被 这个bitmaps拦截掉，从而避免了对底层存储系统的查询压力。

   * **进行实时监控**：当发现Redis的命中率开始急速降低，需要排查访问对象和访问的数据，和运维人员配合，可以设置黑名单限制服务。

### 2、缓存击穿

1. #### 什么是缓存击穿

   key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

   强调的是某个热点key，在redis过期瞬间，有大量请求到达，瞬间击穿缓存，落到后台DB上。

   ![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/缓存击穿.png)

2. #### 解决方案

   * **预先设置热门数据**：在redis高峰访问之前，把一些热门数据提前存入到redis里面，加大这些热门数据key的时长
   * **实时调整**：现场监控哪些数据热门，实时调整key的过期时长
   * **使用锁**：
     * 就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db。
     * 先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX）去set一个mutex key
     * 当操作返回成功时，再进行load db的操作，并回设缓存,最后删除mutex key
     * 当操作返回失败，证明有线程在load db，当前线程睡眠一段时间再重试整个get缓存的方法

   ![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/缓存击穿解决方案使用锁流程图.png)

### 3、缓存雪崩

1. #### 什么是缓存雪崩

   key对应的数据存在，但在redis中过期，此时若有大量并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮。

   缓存雪崩与缓存击穿的区别在于雪崩针对很多key缓存，击穿则是某一个key

   ![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/缓存雪崩.png)

![](/Users/sunwj/Documents/GitHub/JavaGitBook/image/缓存雪崩2.png)

2. #### 解决方案

   * **构建多级缓存架构**：nginx缓存 + redis缓存 +其他缓存（ehcache等）
   * **使用锁或队列**：用加锁或者队列的方式保证来保证不会有大量的线程对数据库一次性进行读写，从而避免失效时大量的并发请求落到底层存储系统上。不适用高并发情况
   * **设置过期标志更新缓存**：记录缓存数据是否过期（设置提前量），如果过期会触发通知另外的线程在后台去更新实际key的缓存。
   * **将缓存失效时间分散开**：比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

### 4、布隆过滤器

1. #### 何为布隆过滤器？

   > 布隆过滤器（Bloom Filter）是1970年由布隆提出的。它实际上是一个很长的二进制向量和一系列随机映射函数。布隆过滤器可以用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都比一般的算法要好的多，缺点是有一定的误识别率和删除困难。 -- *来自百度百科*

2. #### 实现原理

   1. 哈希算法得出的Integer的哈希值最大为：`Integer.MAX_VALUE=2147483647`，意思就是任何一个URL的哈希都会在0~2147483647之间。
   2. 定义一个2147483647长度的byte数组，用来存储集合所有可能的值。为了存储这个byte数组，系统只需要：`2147483647/8/1024/1024=256M`。
   3. 比如：某个URL（X）的哈希是2，那么落到这个byte数组在第二位上就是1，这个byte数组将是：000….00000010，重复的，将这20亿个数全部哈希并落到byte数组中。

   **判断逻辑**

   * 如果byte数组上的第二位是1，那么这个URL（X）可能存在。为什么是可能？因为有可能其它URL因哈              希碰撞哈希出来的也是2，这就是误判。
   * 但是如果这个byte数组上的第二位是0，那么这个URL（X）就一定不存在集合中。

   **多次哈希**

   * 为了减少因哈希碰撞导致的误判概率，可以对这个URL（X）用不同的哈希算法进行N次哈希，得出N个哈希值，落到这个byte数组上，如果这N个位置没有都为1，那么这个URL（X）就一定不存在集合中。

3. #### 布隆过滤器的使用

   Guava框架提供了布隆过滤器的具体实现：BloomFilter，使得开发不用再自己写一套算法的实现。

   BloomFilter提供了几个重载的静态 `create`方法来创建实例：

   ```java
   public static <T> BloomFilter<T> create(Funnel<? super T> funnel, int expectedInsertions, double fpp);
   public static <T> BloomFilter<T> create(Funnel<? super T> funnel, long expectedInsertions, double fpp);
   public static <T> BloomFilter<T> create(Funnel<? super T> funnel, int expectedInsertions);
   public static <T> BloomFilter<T> create(Funnel<? super T> funnel, long expectedInsertions);
   ```

   **最终还是调用**

   ```java
   static <T> BloomFilter<T> create(Funnel<? super T> funnel, long expectedInsertions, double fpp, Strategy strategy);
   // 参数含义：
   // funnel 指定布隆过滤器中存的是什么类型的数据，有：IntegerFunnel，LongFunnel，StringCharsetFunnel。
   // expectedInsertions 预期需要存储的数据量
   // fpp 误判率，默认是0.03。
   ```

   BloomFilter里byte数组的空间大小由 `expectedInsertions`， `fpp`参数决定，见方法:

   ```java
   static long optimalNumOfBits(long n, double p) {
       if (p == 0) {
           p = Double.MIN_VALUE;
       }
       return (long) (-n * Math.log(p) / (Math.log(2) * Math.log(2)));
   }
   ```

   真正的byte数组维护在类：`BitArray`中。

   **使用**

   最后通过：`put`和 `mightContain`方法，添加元素和判断元素是否存在。

   **算法特点：**

   1、因使用哈希判断，时间效率很高。空间效率也是其一大优势。

   2、有误判的可能，需针对具体场景使用。

   3、因为无法分辨哈希碰撞，所以不是很好做删除操作。

   **使用场景：**

   1、黑名单

    2、URL去重 

   3、单词拼写检查 

   4、Key-Value缓存系统的Key校验 

   5、ID校验，比如订单系统查询某个订单ID是否存在，如果不存在就直接返回。