#  Redis常用命令入门5:有序集合类型

- 有序集合类型
```redis
有序集合类型,大家从名字上应该就可以知道,实际上就是在集合类型上加了个有序而已.Redis中的有序集合类型,实际上是在集合类型上,
为每个元素都关联一个分数,有序实际上说的是分数有序,我们根据分数的范围获取集合及其他操作.集合的元素依然是不能够相同的,但是分
数可以相同.
```

- 下面列举有序集合和类型和列表类型的相似处:

①两者都是有序的（废话！）

②两者都可以获得某一范围的元素

- 下面列举区别:

①列表是链表实现的,靠近两边的数据读取极快,而元素过多后获取中间元素的速度则会很慢；有序集合类型使用的散列表和跳跃表（Skip list）
实现的,所以读取哪部分的数据都差不多（时间复杂度是O(logN)）.

②列表中不能简单的调整元素的位置,但是有序集合可以（通过改变分数）.

③有序集合比列表费内存（要存储分数、散列、跳跃表）.

- 下面我们来一起学习命令（这里参数关键字都比较多,所以下面开始列举的命令,关键字都使用大写）.

### 1、增加元素

```redis
ZADD key score member [score member ...]
ZADD命令是向集合中增加元素的命令,往集合中增加分数为score的member,这里也是可以一次增加多个值,返回值是成功增加的元素的个数
,如果member存在,则score会覆盖原有的分数.
```

```redis
127.0.0.1:6379> zadd scoreboard 89 tom
(integer) 1     //添加一个
127.0.0.1:6379> zadd scoreboard 70 peter 100 david
(integer) 2     //添加多个
127.0.0.1:6379> zrange scoreboard 0 -1 withscores
1) "peter"      //带分数输出
2) "70"
3) "tom"
4) "89"
5) "david"
6) "100"
```

**我们发现Peter的分数录入错了,需要修改为76分,这时候我们接着执行下面的命令**

```redis
127.0.0.1:6379> zadd scoreboard 76 peter
(integer) 0         //member存在时,score不一致,会修改score
127.0.0.1:6379> zrange scoreboard 0 -1 withscores
1) "peter"
2) "76"             //70->76
3) "tom"
4) "89"
5) "david"
6) "100"
```

**这里分数不仅仅支持整数,还支持浮点数**

```redis
127.0.0.1:6379> zadd testscore 17e+307 A
(integer) 1
127.0.0.1:6379> zadd testscore 3.3 B
(integer) 1
127.0.0.1:6379> zadd testscore -inf D
(integer) 1
127.0.0.1:6379> zadd testscore +inf C
(integer) 1
127.0.0.1:6379> zrange testscore 0 -1 withscores
1) "D"
2) "-inf"
3) "B"
4) "3.2999999999999998"
5) "A"
6) "1.6999999999999999e+308"
7) "C"
8) "inf"

其中+inf和-inf是正负无穷的意思.
```

### 2、获得元素的分数

```redis
ZSCORE key member
127.0.0.1:6379> zscore scoreboard peter
"76"
```

### 3、获得排名在某个范围的元素列表

```redis
ZRANGE key start stop [WITHSCORE]
ZREVRANGE key start stop [WITHSCORE]
ZRANGE命令会按照元素分数的从小到大顺序返回索引从start到stop之间所有的元素（包含两端）.ZRANGE与LRANGE命令相似,索引
从0开始,负数一样代表从后向前查找（-1是最后一个）.WITHSCORE代表是否加上分数.
```

```redis
127.0.0.1:6379> zrange scoreboard 0 2
1) "peter"
2) "tom"
3) "david"
127.0.0.1:6379> zrange scoreboard 0 -1 withscores
1) "peter"
2) "76"
3) "tom"
4) "89"
5) "david"
6) "100"
127.0.0.1:6379> zrevrange scoreboard 0 -1 withscores
1) "david"
2) "100"
3) "tom"
4) "89"
5) "peter"
6) "76"

ZRANGE命令的时间复杂度为O(longN+m),其中n为有序集合的基数,m为返回的元素个数.如果遇到分数相同的情况,Redis会按照字典顺序
(即”0″<…<”9″<”A”<…<”Z”<”a”<…<”z”这样的顺序)进行排列.如果是中文,也会按照编码之后的字典顺序排序.

ZREVRANGE命令和ZRANGE命令唯一不同的是ZREVRANGE命令是按照分数从大到小给出顺序结果.
```

### 4、获得指定分数范围的元素

```redis
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
这个命令参数很多,但是都很好理解.这个命令用来获取指定分数范围的元素,min是最小值,max是最大值,WITHSCORE还是和上面介绍的一样
LIMIT是为了指定偏移量及数量的,和sql的有点像.offset是偏移量,count是数量.同时这些min和max都是包含的,如果要想不包含,需要使
用“(”符号.
```

```redis
127.0.0.1:6379> zrangebyscore scoreboard 80 100 withscores
1) "tom"
2) "89"
3) "david"
4) "100"
127.0.0.1:6379> zrangebyscore scoreboard (80 100 withscores
1) "tom"
2) "89"
3) "david"
4) "100"
127.0.0.1:6379> zrangebyscore scoreboard 80 (100 withscores
1) "tom"
2) "89"
127.0.0.1:6379> zrangebyscore scoreboard 80 100 withscores limit 1 1
1) "david"
2) "100"
127.0.0.1:6379> zrangebyscore scoreboard 80 100 withscores limit 1 2
1) "david"
2) "100"
127.0.0.1:6379> zrangebyscore scoreboard 60 +inf withscores
1) "peter"
2) "76"
3) "tom"
4) "89"
5) "david"
6) "100"
```

### 5、增加某个元素的分数

```redis
ZINCRBY key incremnet member
127.0.0.1:6379> ZINCRBY scoreboard 2 peter
"78"
127.0.0.1:6379> ZINCRBY scoreboard -5 peter
"73"
这个命令可以增加一个元素的分数,返回值是更改后的分数.这里就不再赘述用法了,和INCRBY命令类似.同样如果不存在会初始为0在增加
,负数即是减小.
```

### 6、获得集合中元素的数量

```redis
ZCARD key
这个命令和SCARD类似,也就不多说了.
```

### 7、获得指定分数范围的元素个数

```redis
ZCOUNT key min max
这里就是获得min和max分数之间的元素数,当然这里也支持“(”符号.
```

### 8、删除一个或多个元素

```redis
ZREM key member [member ...]
返回值是成功删除的元素的个数.
```

```redis
127.0.0.1:6379> zrem scoreboard peter
(integer) 1
127.0.0.1:6379> zrange scoreboard 0 -1
1) "tom"
2) "david"
```

### 9、按照排名范围删除元素

```redis
ZREMRANGEBYRANK key start stop
这个命令按照元素分数从小到大顺序删除指定范围内所有的元素(其实就是先排序然后按照排好的序列的索引删除)并返回删除的元素的数量.
```

### 10、按照分数范围删除元素

```redis
ZREMRANGEBYSCORE key min max
这里就是直接删除分数范围的元素了,这里分数同样支持“(”符号,返回删除数量.
```

### 11、获得元素的排名

```redis
ZRANK key member
ZREVRANK key member
ZRANK命令按照元素分数的从小到大的顺序获得制定元素的排名（第一个从0开始）,ZREVRANK则相反.
```

```redis
127.0.0.1:6379> zrank scoreboard tom
(integer) 0
127.0.0.1:6379> zrank scoreboard david
(integer) 1
```

```redis
最后我们举个实际应用的例子.

我们把wordpress的文章按点击率排序,关系数据库我们是遍历所有的文章排序点击数,如果使用Redis,我们需要一个posts:page.view键
的有序集合类型,然后每个member为文章ID,score为文章的点击量.这样我们就可以用ZREVRANGE命令获取点击量排行榜.

还有一个实际的例子,我们用有序集合类型保存文章的发布时间（时间用UNIX时间及时间的毫秒数）与文章ID,这样我们可以很方便的按时
间来查看文章列表,我们的文章列表应该是用文章发布时间排序而不应该用文章ID排序的.
```
****

作者:UFO

原文:https://github.com/lidawei-ufo/redis/blob/master/redis.md

版权声明:本文为博主原创文章,转载请附上博文链接！


