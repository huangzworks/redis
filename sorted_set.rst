==========
Sorted Set
==========

ZADD
====

.. function:: ZADD key score member

将\ ``member``\ 元素及其\ ``score``\ 值加入到有序集\ ``key``\ 当中。

| \ ``score``\ 值可以是整数值或双精度浮点数。

| 如果\ ``member``\ 已经是有序集的成员，那么更新\ ``member``\ 的\ ``score``\ 值，并通过重新插入\ ``member``\ 元素，来保证\ ``member``\ 在正确的位置上。
| 假如\ ``key``\ 不存在，则创建一个包含\ ``member``\ 元素作成员的有序集。
| 当\ ``key``\ 存在但不是有序集类型时，返回一个错误。

| 对有序集的介绍请移步到页面\ `sorted set <http://redis.io/topics/data-types#sorted-sets>`_\ 查看。

时间复杂度:
    O(1)

返回值:
    | 如果添加元素成功，返回\ ``1``\ 。
    | 如果元素已经是集合的成员，且该元素的\ ``score``\ 值被成功更新，返回\ ``0``\ 。

::

    redis > zadd page_rank 10 google.com
    (integer) 1
    redis > zadd page_rank 9 bing.com
    (integer) 1
    
    redis > zrange page_rank 0 -1 # 显示有序集内所有成员
    1) "bing.com"
    2) "google.com"


ZINTERSTORE
===========

.. function:: ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]

计算给定的一个或多个有序集的交集，其中给定\ ``key``\ 的数量必须以\ ``numkeys``\ 参数指定。

| 默认情况下，结果集中某个成员的\ ``score``\ 值是所有给定集下该成员\ ``score``\ 值的和。
| 关于\ ``WEIGHTS``\ 和\ ``AGGREGATE``\ 选项的描述，参见\ `ZUNION`_\ 命令。

时间复杂度:
    O(N*K)+O(M*log(M))，\ ``N``\ 为给定\ ``key``\ 中基数最小的有序集，\ ``K``\ 为给定有序集的数量，\ ``M``\ 为结果集的基数。

返回值:
    保存到\ ``destination``\ 的结果集的基数。

::
    
    redis > ZADD mid_test 70 "Li Lei"
    (integer) 1
    redis > ZADD mid_test 70 "Han Meimei"
    (integer) 1
    redis > ZADD mid_test 99.5 "Tom"
    (integer) 1

    redis > ZADD fin_test 88 "Li Lei"
    (integer) 1
    redis > ZADD fin_test 75 "Han Meimei"
    (integer) 1
    redis > ZADD fin_test 99.5 "Tom"
    (integer) 1

    redis > ZINTERSTORE sum_point 2 mid_test fin_test
    (integer) 3

    redis > ZRANGE sum_point 0 -1 WITHSCORES  # 显式集合内所有成员及其score值
    1) "Han Meimei"
    2) "145"
    3) "Li Lei"
    4) "158"
    5) "Tom"
    6) "199"


ZREM
====

.. function:: ZREM key member

移除有序集\ ``key``\ 中的成员\ ``member``\ ，如果\ ``member``\ 不是有序集中的成员，那么不执行任何动作。

| 当\ ``key``\ 存在但不是有序集类型时，返回一个错误。

时间复杂度:
    O(log(N))，\ ``N``\ 为有序集的基数。

返回值:
    | 如果\ ``member``\ 被成功移除，返回\ ``1``\ 。
    | 如果\ ``member``\ 不是有序集的成员，返回\ ``0``\ 。

::

    redis > ZADD phone 998 iPh0ne
    (integer) 1
    redis > ZADD phone 999 nokia-5233
    (integer) 1

    redis > ZREM phone iPh0ne # 成功移除
    (integer) 1

    redis > ZREM phone moto-1212  # 移除失败，不存在该成员
    (integer) 0


ZREVRANGEBYSCORE
================

.. function:: ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]

返回有序集\ ``key``\ 中，\ ``score``\ 值介于\ ``max``\ 和\ ``min``\ 之间(包括等于\ ``max``\ 或\ ``min``\ )的所有的成员。有序集成员按\ ``score``\ 值递减(从大到小)的次序排列。

具有相同\ ``score``\ 值的成员按字典序的反序(\ `reverse lexicographical order <http://en.wikipedia.org/wiki/Lexicographical_order#Reverse_lexicographic_order>`_\ )排列。

除了成员按\ ``score``\ 值递减的次序排列这一点外，\ `ZREVRANGEBYSCORE`_\ 和 \ `ZRANGEBYSCORE`_ \ 的作用类似。

时间复杂度:
    O(log(N)+M)，\ ``N``\ 为有序集的基数，\ ``M``\ 为结果集的基数。

返回值:
    带有\ ``score``\ 值(可选)的有序集成员的列表。

::

    redis > ZADD salary 10086 jack
    (integer) 1
    redis > ZADD salary 5000 tom
    (integer) 1
    redis > ZADD salary 7500 peter
    (integer) 1
    redis > ZADD salary 3500 joe
    (integer) 1

    redis > ZREVRANGEBYSCORE salary +inf -inf # 逆序排列所有成员
    1) "jack"
    2) "peter"
    3) "tom"
    4) "joe"

    redis > ZREVRANGEBYSCORE salary 10000 2000 # 逆序排列薪水介于10000和2000之间的成员
    1) "peter"
    2) "tom"
    3) "joe"


ZCARD
=====

.. function:: ZCARD key

返回有序集\ ``key``\ 的基数。

时间复杂度:
    O(1)

返回值:
    | 有序集的基数。
    | 当\ ``key``\ 不存在时，返回\ ``0``\ 。

::

    redis > ZADD salary 2000 tom  # 添加一个成员
    (integer) 1
    redis > ZCARD salary
    (integer) 1

    redis > ZADD salary 5000 jack # 再添加一个成员
    (integer) 1
    redis > ZCARD salary
    (integer) 2

    redis > EXISTS non_exists_key # 对不存在的key进行ZCARD操作
    (integer) 0
    redis > ZCARD non_exists_key
    (integer) 0


ZRANGE
======

.. function:: ZRANGE key start stop [WITHSCORES]

返回有序集\ ``key``\ 中，指定区间内的成员。

| 其中成员的位置按\ ``score``\ 值递增(从小到大)来排序。
| 具有相同\ ``score``\ 值的成员按字典序(\ `lexicographical order <http://en.wikipedia.org/wiki/Lexicographical_order>`_\ )来排列。

| 如果你需要成员按\ ``score``\ 值递减(从大到小)来排列，请使用\ `ZREVRANGE`_\ 。

| 下标参数\ ``start``\ 和\ ``stop``\ 都以\ ``0``\ 为底，也就是说，以\ ``0``\ 指示有序集第一个成员，以\ ``1``\ 指示有序集第二个成员，以此类推。
| 你也可以使用负数下标，以\ ``-1``\ 表示最后一个成员，\ ``-2``\ 表示倒数第二个成员，以此类推。

| 超出范围的下标并不会引起错误。
| 比如说，当\ ``start``\ 的值比有序集的最大下标还要大，或是\ ``start > stop``\ 时，\ `ZRANGE`_\ 操作只是简单地返回一个空列表。
| 另一方面，假如\ ``stop``\ 参数的值比有序集的最大下标还要大，那么Redis将\ ``stop``\ 当作最大下标来处理。

| 可以通过使用\ ``WITHSCORES``\ 选项，来让成员和它的\ ``score``\ 值一并返回，返回列表以\ ``value1,score1, ..., valueN,scoreN``\ 的格式表示。
| 客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

时间复杂度:
    O(log(N)+M)，\ ``N``\ 为有序集的基数，而\ ``M``\ 为返回的结果集的基数。

返回值:
    指定区间内，带有\ ``score``\ 值(可选)的有序集成员的列表。

:: 

   redis > ZADD salary 5000 tom
   (integer) 1
   redis > ZADD salary 10086 boss
   (integer) 1
   redis > ZADD salary 3500 jack
   (integer) 1

   redis > ZRANGE salary 0 -1 WITHSCORES  # 显示整个有序集成员
   1) "jack"
   2) "3500"
   3) "tom"
   4) "5000"
   5) "boss"
   6) "10086"

   redis > ZRANGE salary 1 2 WITHSCORES   # 显示有序集下标区间1至2的成员
   1) "tom"
   2) "5000"
   3) "boss"
   4) "10086"

   redis > ZRANGE salary 0 200000 WITHSCORES  # 测试end下标超出最大下标时的情况
   1) "jack"
   2) "3500"
   3) "tom"
   4) "5000"
   5) "boss"
   6) "10086"

   redis > ZRANGE salary 200000 3000000 WITHSCORES   # 测试当给定区间不存在于有序集时的情况 
   (empty list or set)


ZREMRANGEBYRANK
===============

.. function:: ZREMRANGEBYRANK key start stop

移除有序集\ ``key``\ 中，指定排名(rank)区间内的所有成员。

| 区间分别以下标参数\ ``start``\ 和\ ``stop``\ 指出，包含\ ``start``\ 和\ ``stop``\ 在内。
| 下标参数\ ``start``\ 和\ ``stop``\ 都以\ ``0``\ 为底，也就是说，以\ ``0``\ 指示有序集第一个成员，以\ ``1``\ 指示有序集第二个成员，以此类推。
| 你也可以使用负数下标，以\ ``-1``\ 表示最后一个成员，\ ``-2``\ 表示倒数第二个成员，以此类推。

时间复杂度:
    O(log(N)+M)，\ ``N``\ 为有序集的基数，而\ ``M``\ 为移除成员。

返回值:
    移除成员的个数。

::

    redis 127.0.0.1:6379> ZADD salary 2000 jack
    (integer) 1
    redis 127.0.0.1:6379> ZADD salary 5000 tom
    (integer) 1
    redis 127.0.0.1:6379> ZADD salary 3500 peter
    (integer) 1

    redis 127.0.0.1:6379> ZREMRANGEBYRANK salary 0 1    # 移除下标0至1区间内的成员
    (integer) 2

    redis 127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES # 有序集只剩下一个成员
    1) "tom"
    2) "5000"


ZREVRANK
========

.. function:: ZREVRANK key member

返回有序集\ ``key``\ 中成员\ ``member``\ 的排名。其中有序集成员按\ ``score``\ 值递减(从大到小)排序。

排名以\ ``0``\ 为底，也就是说，\ ``score``\ 值最大的成员排名为\ ``0``\ 。

使用\ `ZRANK`_\ 可以获得成员按\ ``score``\ 值递增(从小到大)排列的排名。

时间复杂度:
    O(log(N))

返回值:
    | 如果\ ``member``\ 是有序集\ ``key``\ 的成员，返回\ ``member``\ 的排名。
    | 如果\ ``member``\ 不是有序集\ ``key``\ 的成员，返回\ ``nil``\ 。

::

    redis 127.0.0.1:6379> ZADD salary 2000 jack
    (integer) 1
    redis 127.0.0.1:6379> ZADD salary 5000 tom
    (integer) 1
    redis 127.0.0.1:6379> ZADD salary 3500 peter
    (integer) 1

    redis 127.0.0.1:6379> ZREVRANK salary peter # peter的工资排第二
    (integer) 1
    redis 127.0.0.1:6379> ZREVRANK salary tom   # tom的工资最高
    (integer) 0


ZCOUNT
======

.. function:: ZCOUNT key min max

返回有序集\ ``key``\ 中，\ ``score``\ 值在\ ``min``\ 和\ ``max``\ 之间(包括等于\ ``min``\ 或\ ``max``\ )的成员。

参数\ ``min``\ 和\ ``max``\ 的作用和\ `ZRANGEBYSCORE`_\ 中描述的一样。

时间复杂度:
    O(log(N)+M)

返回值:
    \ ``score``\ 值在\ ``min``\ 和\ ``max``\ 之间的成员的数量。

::

    redis 127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES # 显示所有成员及其score值
    1) "jack"
    2) "2000"
    3) "peter"
    4) "3500"
    5) "tom"
    6) "5000"

    redis 127.0.0.1:6379> ZCOUNT salary 2000 5000   # 计算薪水在2000-5000之间的人数
    (integer) 3

    redis 127.0.0.1:6379> ZCOUNT salary 3000 5000   # 计算薪水在3000-5000之间的人数
    (integer) 2


ZRANGEBYSCORE
=============

.. function:: ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]


