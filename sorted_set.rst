==========
Sorted Set
==========

ZADD
====

.. function:: ZADD key score member

将member元素及其分值score加入到有序集key当中。

| 分值score可以是整数值或双精度浮点数。

| 如果member已经是有序集的成员，那么更新member的score，并通过重新插入member元素，来保证member在正确的位置上。
| 假如key不存在，则创建一个包含member元素作成员的有序集。
| 当key存在但不是有序集类型时，返回一个错误。

| 对有序集的介绍请移步到页面\ `sorted set <http://redis.io/topics/data-types#sorted-sets>`_\ 查看。

时间复杂度:
    O(1)

返回值:
    | 如果添加元素成功，返回1。
    | 如果元素已经是集合的成员，且该元素的score值被成功更新，返回0。

::

    redis 127.0.0.1:6379> zadd page_rank 10 google.com
    (integer) 1
    redis 127.0.0.1:6379> zadd page_rank 9 bing.com
    (integer) 1
    
    redis 127.0.0.1:6379> zrange page_rank 0 -1 # 显示有序集内所有成员
    1) "bing.com"
    2) "google.com"


ZINTERSTORE
===========

.. function:: ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]

计算给定的一个或多个有序集的交集，其中给定key的数量必须以numkeys参数指定。

| 默认情况下，结果集中某个元素的score值是所有给定集下该元素score值的和。
| 关于WEIGHTS和AGGREGATE选项的描述，参见\ `ZUNION`_\ 命令。

时间复杂度:
    O(N*K)+O(M*log(M))，N为给定key中基数最小的有序集，K为给定有序集的数量，M为结果集的基数。

返回值:
    保存到destination的结果集的基数。

::
    
    redis 127.0.0.1:6379> ZADD mid_test 70 "Li Lei"
    (integer) 1
    redis 127.0.0.1:6379> ZADD mid_test 70 "Han Meimei"
    (integer) 1
    redis 127.0.0.1:6379> ZADD mid_test 99.5 "Tom"
    (integer) 1

    redis 127.0.0.1:6379> ZADD fin_test 88 "Li Lei"
    (integer) 1
    redis 127.0.0.1:6379> ZADD fin_test 75 "Han Meimei"
    (integer) 1
    redis 127.0.0.1:6379> ZADD fin_test 99.5 "Tom"
    (integer) 1

    redis 127.0.0.1:6379> ZINTERSTORE sum_point 2 mid_test fin_test
    (integer) 3

    redis 127.0.0.1:6379> ZRANGE sum_point 0 -1 WITHSCORES  # 显式集合内所有成员及其score值
    1) "Han Meimei"
    2) "145"
    3) "Li Lei"
    4) "158"
    5) "Tom"
    6) "199"


ZREM
====

.. function:: ZREM key member

移除有序集key中的成员member，如果member不是有序集中的成员，那么不执行任何动作。

| 当key存在但不是有序集类型时，返回一个错误。

时间复杂度:
    O(log(N))，N为有序集的基数。

返回值:
    | 如果member被成功移除，返回1。
    | 如果member不是有序集的成员，返回0。

::

    redis 127.0.0.1:6379> ZADD phone 998 iPh0ne
    (integer) 1
    redis 127.0.0.1:6379> ZADD phone 999 nokia-5233
    (integer) 1

    redis 127.0.0.1:6379> ZREM phone iPh0ne # 成功移除
    (integer) 1

    redis 127.0.0.1:6379> ZREM phone moto-1212  # 移除失败，不存在该成员
    (integer) 0


ZREVRANGEBYSCORE
================

.. function:: ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]

返回有序集key中，所有score值介于max和min之间(包括等于max或min)的所有的元素。元素以递减(从大到小)的次序排列。

具有相同score值的元素以字典序的反序(\ `reverse lexicographical order <http://en.wikipedia.org/wiki/Lexicographical_order#Reverse_lexicographic_order>`_\ )排列。

除了元素以递减的次序排序这一点外，\ `ZREVRANGEBYSCORE`_\ 和 \ `ZRANGEBYSCORE`_ \ 的作用类似。

时间复杂度:
    O(log(N)+M)，N为有序集的基数，M为结果集的基数。

返回值:
    带有score值(可选)的有序集成员的列表。

::

    redis 127.0.0.1:6379> ZADD salary 10086 jack
    (integer) 1
    redis 127.0.0.1:6379> ZADD salary 5000 tom
    (integer) 1
    redis 127.0.0.1:6379> ZADD salary 7500 peter
    (integer) 1
    redis 127.0.0.1:6379> ZADD salary 3500 joe
    (integer) 1

    redis 127.0.0.1:6379> ZREVRANGEBYSCORE salary +inf -inf # 逆序排列所有成员
    1) "jack"
    2) "peter"
    3) "tom"
    4) "joe"

    redis 127.0.0.1:6379> ZREVRANGEBYSCORE salary 10000 2000 # 逆序排列薪水介于10000和2000之间的成员
    1) "peter"
    2) "tom"
    3) "joe"


ZCARD
=====

.. function:: ZCARD key

返回有序集key的基数。

时间复杂度:
    O(1)

返回值:
    | 有序集的基数。
    | 当key不存在时，返回0。

::

    redis 127.0.0.1:6379> ZADD salary 2000 tom  # 添加一个元素
    (integer) 1
    redis 127.0.0.1:6379> ZCARD salary
    (integer) 1

    redis 127.0.0.1:6379> ZADD salary 5000 jack # 再添加一个元素
    (integer) 1
    redis 127.0.0.1:6379> ZCARD salary
    (integer) 2

    redis 127.0.0.1:6379> EXISTS non_exists_key # 对不存在的key进行ZCARD操作
    (integer) 0
    redis 127.0.0.1:6379> ZCARD non_exists_key
    (integer) 0


ZRANGE
======

.. function:: ZRANGE key start stop [WITHSCORES]

返回有序集key中，指定区间内的成员。

| 其中成员的位置按score值递增(从小到大)来排序。
| 具有相同score值的成员按字典序(\ `lexicographical order <http://en.wikipedia.org/wiki/Lexicographical_order>`_\ )来排序。

| 如果你需要成员按score值递减(从大到小)来排序，请使用\ `ZREVRANGE`_\ 。

| 下标start和stop参数都以0为底，也即是，以0指示有序集第一个元素，以1指示有序集第二个元素，依此类推。
| 你也可以使用负数下标，以-1表示最后一个元素，-2表示倒数第二个元素，以此类推。

| 超出范围的下标并不会引起错误。
| 比如说，当start的值比有序集的最大下标还要大，或是start > stop时，\ `ZRANGE`_\ 操作只是简单地返回一个空列表。
| 另一方面，假如stop参数的值比有序集的最大下标还要大，那么Redis将stop当作最大下标来处理。

| 可以通过使用WITHSCORES选项，来让成员和它的score值一并返回，返回列表以value1,score1, ..., valueN,scoreN的格式表示。
| 客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

时间复杂度:
    O(log(N)+M)，N为有序集的基数，而M为返回的结果集的基数。

返回值:
    指定区间内，带有score值(可选)的有序集成员的列表。

::

   redis 127.0.0.1:6379> ZADD salary 5000 tom
   (integer) 1
   redis 127.0.0.1:6379> ZADD salary 10086 boss
   (integer) 1
   redis 127.0.0.1:6379> ZADD salary 3500 jack
   (integer) 1

   redis 127.0.0.1:6379> ZRANGE salary 0 -1 WITHSCORES  # 显示整个有序集成员
   1) "jack"
   2) "3500"
   3) "tom"
   4) "5000"
   5) "boss"
   6) "10086"

   redis 127.0.0.1:6379> ZRANGE salary 1 2 WITHSCORES   # 显示有序集下标区间1至2的成员
   1) "tom"
   2) "5000"
   3) "boss"
   4) "10086"

   redis 127.0.0.1:6379> ZRANGE salary 0 200000 WITHSCORES  # 测试end下标超出最大下标时的情况
   1) "jack"
   2) "3500"
   3) "tom"
   4) "5000"
   5) "boss"
   6) "10086"

   redis 127.0.0.1:6379> ZRANGE salary 200000 3000000 WITHSCORES    # 测试当给定区间不存在于有序集时的情况
   (empty list or set)



