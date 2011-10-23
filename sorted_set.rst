.. _sorted_set_struct:

有序集(Sorted Set)
*******************

.. _zadd:

ZADD
====

.. function:: ZADD key score member [[score member] [score member] ...]

将一个或多个\ ``member``\ 元素及其\ ``score``\ 值加入到有序集\ ``key``\ 当中。

如果某个\ ``member``\ 已经是有序集的成员，那么更新这个\ ``member``\ 的\ ``score``\ 值，并通过重新插入这个\ ``member``\ 元素，来保证该\ ``member``\ 在正确的位置上。

\ ``score``\ 值可以是整数值或双精度浮点数。

如果\ ``key``\ 不存在，则创建一个空的有序集并执行\ `ZADD`_\ 操作。

当\ ``key``\ 存在但不是有序集类型时，返回一个错误。

对有序集的更多介绍请参见\ `sorted set <http://redis.io/topics/data-types#sorted-sets>`_\ 。

**时间复杂度:**
    O(M*log(N))，\ ``N``\ 是有序集的基数，\ ``M``\ 为成功添加的新成员的数量。

**返回值:**
    被成功添加的\ *新*\ 成员的数量，不包括那些被更新的、已经存在的成员。

.. note:: 在Redis2.4版本以前，\ `ZADD`_\ 每次只能添加一个元素。

::

    # 添加单个元素

    redis> ZADD page_rank 10 google.com
    (integer) 1

    # 添加多个元素

    redis> ZADD page_rank 9 baidu.com 8 bing.com
    (integer) 2

    redis> ZRANGE page_rank 0 -1 WITHSCORES
    1) "bing.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"

    # 添加已存在元素，且 score 值不变

    redis> ZADD page_rank 10 google.com
    (integer) 0

    redis> ZRANGE page_rank 0 -1 WITHSCORES  # 没有改变
    1) "bing.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"

    # 添加已存在元素，但是改变 score 值

    redis> ZADD page_rank 6 bing.com
    (integer) 0

    redis> ZRANGE page_rank 0 -1 WITHSCORES  # bing.com 元素的score值被改变
    1) "bing.com"
    2) "6"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"


.. _zrem:

ZREM
=====

.. function:: ZREM key member [member ...]

移除有序集\ ``key``\ 中的一个或多个成员，不存在的成员将被忽略。

当\ ``key``\ 存在但不是有序集类型时，返回一个错误。

**时间复杂度:**
    O(M*log(N))，\ ``N``\ 为有序集的基数，\ ``M``\ 为被成功移除的成员的数量。

**返回值:**
    被成功移除的成员的数量，不包括被忽略的成员。

.. note:: 在Redis2.4版本以前，\ `ZREM`_\ 每次只能删除一个元素。

::

    # 测试数据

    redis> ZRANGE page_rank 0 -1 WITHSCORES
    1) "bing.com"
    2) "8"
    3) "baidu.com"
    4) "9"
    5) "google.com"
    6) "10"

    # 移除单个元素

    redis> ZREM page_rank google.com
    (integer) 1

    redis> ZRANGE page_rank 0 -1 WITHSCORES
    1) "bing.com"
    2) "8"
    3) "baidu.com"
    4) "9"

    # 移除多个元素

    redis> ZREM page_rank baidu.com bing.com
    (integer) 2

    redis> ZRANGE page_rank 0 -1 WITHSCORES
    (empty list or set)

    # 移除不存在元素

    redis> ZREM page_rank non-exists-element
    (integer) 0


.. _zcard:

ZCARD
======

.. function:: ZCARD key

返回有序集\ ``key``\ 的基数。

**时间复杂度:**
    O(1)

**返回值:**
    | 当\ ``key``\ 存在且是有序集类型时，返回有序集的基数。
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


.. _zcount:

ZCOUNT
=======

.. function:: ZCOUNT key min max

返回有序集\ ``key``\ 中，\ ``score``\ 值在\ ``min``\ 和\ ``max``\ 之间(默认包括\ ``score``\ 值等于\ ``min``\ 或\ ``max``\ )的成员。

关于参数\ ``min``\ 和\ ``max``\ 的详细使用方法，请参考\ `ZRANGEBYSCORE`_\ 命令。

**时间复杂度:**
    O(log(N)+M)，\ ``N``\ 为有序集的基数，\ ``M``\ 为值在\ ``min``\ 和\ ``max``\ 之间的元素的数量。

**返回值:**
    \ ``score``\ 值在\ ``min``\ 和\ ``max``\ 之间的成员的数量。

::

    redis> ZRANGE salary 0 -1 WITHSCORES # 显示所有成员及其score值
    1) "jack"
    2) "2000"
    3) "peter"
    4) "3500"
    5) "tom"
    6) "5000"

    redis> ZCOUNT salary 2000 5000   # 计算薪水在2000-5000之间的人数
    (integer) 3

    redis> ZCOUNT salary 3000 5000   # 计算薪水在3000-5000之间的人数
    (integer) 2


.. _zscore:

ZSCORE
======

.. function:: ZSCORE key member

返回有序集\ ``key``\ 中，成员\ ``member``\ 的\ ``score``\ 值。

如果\ ``member``\ 元素不是有序集\ ``key``\ 的成员，或\ ``key``\ 不存在，返回\ ``nil``\ 。

**时间复杂度:**
    O(1)

**返回值:**
    \ ``member``\ 成员的\ ``score``\ 值，以字符串形式表示。

::
    
    redis> ZRANGE salary 0 -1 WITHSCORES # 显示所有成员及其score值
    1) "tom"
    2) "2000"
    3) "peter"
    4) "3500"
    5) "jack"
    6) "5000"

    redis> ZSCORE salary peter   # 注意返回值是字符串
    "3500"


.. _zincrby:

ZINCRBY
========

.. function:: ZINCRBY key increment member

为有序集\ ``key``\ 的成员\ ``member``\ 的\ ``score``\ 值加上增量\ ``increment``\ 。

你也可以通过传递一个负数值\ ``increment``\ ，让\ ``score``\ 减去相应的值，比如\ ``ZINCRBY key -5 member``\ ，就是让\ ``member``\ 的\ ``score``\ 值减去\ ``5``\ 。

当\ ``key``\ 不存在，或\ ``member``\ 不是\ ``key``\ 的成员时，\ ``ZINCRBY key increment member``\ 等同于\ ``ZADD key increment member``\ 。

当\ ``key``\ 不是有序集类型时，返回一个错误。

\ ``score``\ 值可以是整数值或双精度浮点数。

**时间复杂度:**
    O(log(N))

**返回值:**
    \ ``member``\ 成员的新\ ``score``\ 值，以字符串形式表示。

::

    redis> ZSCORE salary tom 
    "2000"

    redis> ZINCRBY salary 2000 tom   # tom加薪啦！
    "4000"


.. _zrange:

ZRANGE
=======

.. function:: ZRANGE key start stop [WITHSCORES]

返回有序集\ ``key``\ 中，指定区间内的成员。

其中成员的位置按\ ``score``\ 值递增(从小到大)来排序。

具有相同\ ``score``\ 值的成员按字典序(\ `lexicographical order`_\ )来排列。

如果你需要成员按\ ``score``\ 值递减(从大到小)来排列，请使用\ `ZREVRANGE`_\ 命令。

| 下标参数\ ``start``\ 和\ ``stop``\ 都以\ ``0``\ 为底，也就是说，以\ ``0``\ 表示有序集第一个成员，以\ ``1``\ 表示有序集第二个成员，以此类推。
| 你也可以使用负数下标，以\ ``-1``\ 表示最后一个成员，\ ``-2``\ 表示倒数第二个成员，以此类推。

| 超出范围的下标并不会引起错误。
| 比如说，当\ ``start``\ 的值比有序集的最大下标还要大，或是\ ``start > stop``\ 时，\ `ZRANGE`_\ 命令只是简单地返回一个空列表。
| 另一方面，假如\ ``stop``\ 参数的值比有序集的最大下标还要大，那么Redis将\ ``stop``\ 当作最大下标来处理。

| 可以通过使用\ ``WITHSCORES``\ 选项，来让成员和它的\ ``score``\ 值一并返回，返回列表以\ ``value1,score1, ..., valueN,scoreN``\ 的格式表示。
| 客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

**时间复杂度:**
    O(log(N)+M)，\ ``N``\ 为有序集的基数，而\ ``M``\ 为结果集的基数。

**返回值:**
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


.. _zrevrange:

ZREVRANGE
===========

.. function:: ZREVRANGE key start stop [WITHSCORES]

返回有序集\ ``key``\ 中，指定区间内的成员。

| 其中成员的位置按\ ``score``\ 值递减(从大到小)来排列。
| 具有相同\ ``score``\ 值的成员按字典序的反序(\ `reverse lexicographical order`_\ )排列。

除了成员按\ ``score``\ 值递减的次序排列这一点外，\ `ZREVRANGE`_\ 命令的其他方面和\ `ZRANGE`_\ 命令一样。

**时间复杂度:**
    O(log(N)+M)，\ ``N``\ 为有序集的基数，而\ ``M``\ 为结果集的基数。

**返回值:**
    指定区间内，带有\ ``score``\ 值(可选)的有序集成员的列表。

::

    redis> ZRANGE salary 0 -1 WITHSCORES # 递增排列
    1) "peter"
    2) "3500"
    3) "tom"
    4) "4000"
    5) "jack"
    6) "5000"

    redis> ZREVRANGE salary 0 -1 WITHSCORES  # 递减排列
    1) "jack"
    2) "5000"
    3) "tom"
    4) "4000"
    5) "peter"
    6) "3500"


.. _zrangebyscore:

ZRANGEBYSCORE
==============

.. function:: ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]

返回有序集\ ``key``\ 中，所有\ ``score``\ 值介于\ ``min``\ 和\ ``max``\ 之间(包括等于\ ``min``\ 或\ ``max``\ )的成员。有序集成员按\ ``score``\ 值递增(从小到大)次序排列。

具有相同\ ``score``\ 值的成员按字典序(\ `lexicographical order`_\ )来排列(该属性是有序集提供的，不需要额外的计算)。

可选的\ ``LIMIT``\ 参数指定返回结果的数量及区间(就像SQL中的\ ``SELECT LIMIT offset, count``\ )，注意当\ ``offset``\ 很大时，定位\ ``offset``\ 的操作可能需要遍历整个有序集，此过程最坏复杂度为O(N)时间。

| 可选的\ ``WITHSCORES``\ 参数决定结果集是单单返回有序集的成员，还是将有序集成员及其\ ``score``\ 值一起返回。
| 该选项自Redis 2.0版本起可用。

**区间及无限**

\ ``min``\ 和\ ``max``\ 可以是\ ``-inf``\ 和\ ``+inf``\ ，这样一来，你就可以在不知道有序集的最低和最高\ ``score``\ 值的情况下，使用\ `ZRANGEBYSCORE`_\ 这类命令。

默认情况下，区间的取值使用\ `闭区间 <http://zh.wikipedia.org/wiki/%E5%8D%80%E9%96%93>`_\ (小于等于或大于等于)，你也可以通过给参数前增加\ ``(``\ 符号来使用可选的\ `开区间 <http://zh.wikipedia.org/wiki/%E5%8D%80%E9%96%93>`_\ (小于或大于)。

举个例子：

:: 

    ZRANGEBYSCORE zset (1 5

返回所有符合条件\ ``1 < score <= 5``\ 的成员；

::

    ZRANGEBYSCORE zset (5 (10

返回所有符合条件\ ``5 < score < 10``\ 的成员。

**时间复杂度:**
    O(log(N)+M)，\ ``N``\ 为有序集的基数，\ ``M``\ 为被结果集的基数。

**返回值:**
    指定区间内，带有\ ``score``\ 值(可选)的有序集成员的列表。

::

    redis> ZADD salary 2500 jack
    (integer) 0
    redis> ZADD salary 5000 tom
    (integer) 0
    redis> ZADD salary 12000 peter
    (integer) 0

    redis> ZRANGEBYSCORE salary -inf +inf    # 显示整个有序集
    1) "jack"
    2) "tom"
    3) "peter"

    redis> ZRANGEBYSCORE salary -inf +inf WITHSCORES # 显示整个有序集及成员的score值
    1) "jack"
    2) "2500"
    3) "tom"
    4) "5000"
    5) "peter"
    6) "12000"

    redis> ZRANGEBYSCORE salary -inf 5000 WITHSCORES # 显示工资<=5000的所有成员
    1) "jack"
    2) "2500"
    3) "tom"
    4) "5000"

    redis> ZRANGEBYSCORE salary (5000 400000 # 显示工资大于5000小于400000的成员
    1) "peter"


.. _zrevrangebyscore:

ZREVRANGEBYSCORE
=================

.. function:: ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]

返回有序集\ ``key``\ 中，\ ``score``\ 值介于\ ``max``\ 和\ ``min``\ 之间(默认包括等于\ ``max``\ 或\ ``min``\ )的所有的成员。有序集成员按\ ``score``\ 值递减(从大到小)的次序排列。

具有相同\ ``score``\ 值的成员按字典序的反序(\ `reverse lexicographical order`_\ )排列。

除了成员按\ ``score``\ 值递减的次序排列这一点外，\ `ZREVRANGEBYSCORE`_\ 命令的其他方面和\ `ZRANGEBYSCORE`_\ 命令一样。

**时间复杂度:**
    O(log(N)+M)，\ ``N``\ 为有序集的基数，\ ``M``\ 为结果集的基数。

**返回值:**
    指定区间内，带有\ ``score``\ 值(可选)的有序集成员的列表。

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


.. _zrank:

ZRANK
=======

.. function:: ZRANK key member

返回有序集\ ``key``\ 中成员\ ``member``\ 的排名。其中有序集成员按\ ``score``\ 值递增(从小到大)顺序排列。

排名以\ ``0``\ 为底，也就是说，\ ``score``\ 值最小的成员排名为\ ``0``\ 。

使用\ `ZREVRANK`_\ 命令可以获得成员按\ ``score``\ 值递减(从大到小)排列的排名。

**时间复杂度:**
    O(log(N))

**返回值:**
    | 如果\ ``member``\ 是有序集\ ``key``\ 的成员，返回\ ``member``\ 的排名。
    | 如果\ ``member``\ 不是有序集\ ``key``\ 的成员，返回\ ``nil``\ 。

::

    redis> ZRANGE salary 0 -1 WITHSCORES # 显示所有成员及其score值
    1) "peter"
    2) "3500"
    3) "tom"
    4) "4000"
    5) "jack"
    6) "5000"

    redis> ZRANK salary tom  # 显示tom的薪水排名，第二
    (integer) 1


.. _zrevrank:

ZREVRANK
=========

.. function:: ZREVRANK key member

返回有序集\ ``key``\ 中成员\ ``member``\ 的排名。其中有序集成员按\ ``score``\ 值递减(从大到小)排序。

排名以\ ``0``\ 为底，也就是说，\ ``score``\ 值最大的成员排名为\ ``0``\ 。

使用\ `ZRANK`_\ 命令可以获得成员按\ ``score``\ 值递增(从小到大)排列的排名。

**时间复杂度:**
    O(log(N))

**返回值:**
    | 如果\ ``member``\ 是有序集\ ``key``\ 的成员，返回\ ``member``\ 的排名。
    | 如果\ ``member``\ 不是有序集\ ``key``\ 的成员，返回\ ``nil``\ 。

::

    redis> ZADD salary 2000 jack
    (integer) 1
    redis> ZADD salary 5000 tom
    (integer) 1
    redis> ZADD salary 3500 peter
    (integer) 1

    redis> ZREVRANK salary peter # peter的工资排第二
    (integer) 1
    redis> ZREVRANK salary tom   # tom的工资最高
    (integer) 0


.. _zremrangebyrank:

ZREMRANGEBYRANK
================

.. function:: ZREMRANGEBYRANK key start stop

移除有序集\ ``key``\ 中，指定排名(rank)区间内的所有成员。

区间分别以下标参数\ ``start``\ 和\ ``stop``\ 指出，包含\ ``start``\ 和\ ``stop``\ 在内。

| 下标参数\ ``start``\ 和\ ``stop``\ 都以\ ``0``\ 为底，也就是说，以\ ``0``\ 表示有序集第一个成员，以\ ``1``\ 表示有序集第二个成员，以此类推。
| 你也可以使用负数下标，以\ ``-1``\ 表示最后一个成员，\ ``-2``\ 表示倒数第二个成员，以此类推。

**时间复杂度:**
    O(log(N)+M)，\ ``N``\ 为有序集的基数，而\ ``M``\ 为被移除成员的数量。

**返回值:**
    被移除成员的数量。

::

    redis> ZADD salary 2000 jack
    (integer) 1
    redis> ZADD salary 5000 tom
    (integer) 1
    redis> ZADD salary 3500 peter
    (integer) 1

    redis> ZREMRANGEBYRANK salary 0 1    # 移除下标0至1区间内的成员
    (integer) 2

    redis> ZRANGE salary 0 -1 WITHSCORES # 有序集只剩下一个成员
    1) "tom"
    2) "5000"


.. _zremrangebyscore:

ZREMRANGEBYSCORE
=================

.. function:: ZREMRANGEBYSCORE key min max

移除有序集\ ``key``\ 中，所有\ ``score``\ 值介于\ ``min``\ 和\ ``max``\ 之间(包括等于\ ``min``\ 或\ ``max``\ )的成员。

自版本2.1.6开始，\ ``score``\ 值等于\ ``min``\ 或\ ``max``\ 的成员也可以不包括在内，详情请参见\ `ZRANGEBYSCORE`_\ 命令。

**时间复杂度:**
    O(log(N)+M)，\ ``N``\ 为有序集的基数，而\ ``M``\ 为被移除成员的数量。

**返回值:**
    被移除成员的数量。

::
    
    redis> ZRANGE salary 0 -1 WITHSCORES # 显示有序集内所有成员及其score值
    1) "tom"
    2) "2000"
    3) "peter"
    4) "3500"
    5) "jack"
    6) "5000"

    redis> ZREMRANGEBYSCORE salary 1500 3500 # 解雇所有薪水在1500到3500内的员工
    (integer) 2

    redis> ZRANGE salary 0 -1 WITHSCORES # 剩下的有序集成员
    1) "jack"
    2) "5000"


.. _zinterstore:

ZINTERSTORE
=============

.. function:: ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]

计算给定的一个或多个有序集的交集，其中给定\ ``key``\ 的数量必须以\ ``numkeys``\ 参数指定，并将该交集(结果集)储存到\ ``destination``\ 。

默认情况下，结果集中某个成员的\ ``score``\ 值是所有给定集下该成员\ ``score``\ 值之\ *和*\ 。

关于\ ``WEIGHTS``\ 和\ ``AGGREGATE``\ 选项的描述，参见\ `ZUNIONSTORE`_\ 命令。

**时间复杂度:**
    O(N*K)+O(M*log(M))，\ ``N``\ 为给定\ ``key``\ 中基数最小的有序集，\ ``K``\ 为给定有序集的数量，\ ``M``\ 为结果集的基数。

**返回值:**
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


.. _zunionstore:

ZUNIONSTORE
============

.. function:: ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]

计算给定的一个或多个有序集的并集，其中给定\ ``key``\ 的数量必须以\ ``numkeys``\ 参数指定，并将该并集(结果集)储存到\ ``destination``\ 。

默认情况下，结果集中某个成员的\ ``score``\ 值是所有给定集下该成员\ ``score``\ 值之\ *和*\ 。

**WEIGHTS**

使用\ ``WEIGHTS``\ 选项，你可以为\ *每个*\ 给定有序集\ *分别*\ 指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的\ ``score``\ 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子。

如果没有指定\ ``WEIGHTS``\ 选项，乘法因子默认设置为\ ``1``\ 。

**AGGREGATE**

使用\ ``AGGREGATE``\ 选项，你可以指定并集的结果集的聚合方式。

默认使用的参数\ ``SUM``\ ，可以将所有集合中某个成员的\ ``score``\ 值之\ *和*\ 作为结果集中该成员的\ ``score``\ 值；使用参数\ ``MIN``\ ，可以将所有集合中某个成员的\ *最小*\ \ ``score``\ 值作为结果集中该成员的\ ``score``\ 值；而参数\ ``MAX``\ 则是将所有集合中某个成员的\ *最大*\ \ ``score``\ 值作为结果集中该成员的\ ``score``\ 值。

**时间复杂度:**
    O(N)+O(M log(M))，\ ``N``\ 为给定有序集基数的总和，\ ``M``\ 为结果集的基数。

**返回值:**
    保存到\ ``destination``\ 的结果集的基数。

::

    redis> ZRANGE programmer 0 -1 WITHSCORES
    1) "peter"
    2) "2000"
    3) "jack"
    4) "3500"
    5) "tom"
    6) "5000"

    redis> ZRANGE manager 0 -1 WITHSCORES
    1) "herry"
    2) "2000"
    3) "mary"
    4) "3500"
    5) "bob"
    6) "4000"

    redis> ZUNIONSTORE salary 2 programmer manager WEIGHTS 1 3   # 公司决定加薪。。。除了程序员。。。
    (integer) 6

    redis> ZRANGE salary 0 -1 WITHSCORES
    1) "peter"
    2) "2000"
    3) "jack"
    4) "3500"
    5) "tom"
    6) "5000"
    7) "herry"
    8) "6000"
    9) "mary"
    10) "10500"
    11) "bob"
    12) "12000"

.. _reverse lexicographical order: http://en.wikipedia.org/wiki/Lexicographical_order#Reverse_lexicographic_order
.. _lexicographical order: http://en.wikipedia.org/wiki/Lexicographical_order
